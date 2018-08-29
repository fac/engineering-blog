---
:title: How we run 4 hours of tests in under 4 minutes
:created_at: 2016-05-05 16:00:00.000000000 +00:00
:kind: article
:tags:
- Jenkins
- CI
- Testing
:author: donal
:slug: four-hours-of-tests-in-four-minutes
---

<% excerpt do %>
Here at FreeAgent we have a test suite that contains over 21,000 individual RSpec examples. Currently it takes approximately 4 hours to run in a single process.

Here’s how we’ve tuned our test suite and CI system (Jenkins) to run them in under 4 minutes.
<% end %>

# 1. Parallelise

The first step is to run the specs in parallel.

## Test queue

We use [test-queue](https://github.com/tmm1/test-queue), a parallel test runner by Aman Gupta.

It uses a centralised queue and allows you to distribute the test jobs across multiple nodes. On each node it forks multiple worker processes. These workers then poll the central queue for jobs.

In our case, as we use RSpec, each job is an RSpec example group.

Test-queue has a couple of advantages:

1. We can easily add extra worker nodes to increase the parallelisation
2. It uses a central queue and sorts jobs by previous run times, so each worker is fully utilised for the entire test run

This second point is critical — because you don’t need to pre-slice the workload, you don’t have to worry about rebalancing it when you add nodes. You can also add nodes with different performance characteristics.

## Fix leaky specs

Because test queue distributes the specs on demand to the workers, they will run in a different order each time. This means that if you have specs that leak state, you can start to see random failures where there are unwanted dependencies between different specs.

You’ll need to find and squash these before your specs can run reliably. This is a time consuming task, but a good way to track them down is to run all the specs in a single process in forward and reverse order. This should show up anywhere that spec A depends on spec B or the reverse.

We run a nightly Jenkins task to run the specs in a single process that helps to spot any new issues that have been introduced.

## Single pool of jobs

To take full advantage of test-queue, we needed a single pool of jobs to run. However, we had a mixture of RSpec, Cucumber and [Teaspoon](https://github.com/modeset/teaspoon) tests.

We converted the Cucumber specs to [Turnip](https://github.com/jnicklas/turnip) which allows them to be run with RSpec. This mainly involved converting the steps to the Turnip format (which is much nicer than Cucumber’s anyway).

The Teaspoon specs were wrapped up with an RSpec matcher that runs a Teaspoon suite and ensures that no failures are reported.

## Lots of workers

Now that we have a homogenous pool of RSpec example groups and a distributed test runner, we can just add more nodes and speed the test suite up!

We run the specs across 98 worker processes on 7 Jenkins nodes, with each node running 14 workers processes.

We run the FreeAgent application from two data centres, one serving production traffic and the other as a backup. We run Jenkins in the backup data centre which allows us to utilise our spare capacity and avoids impacting production services.

## Reduce the size of the largest jobs

When you start adding lots of workers, you become limited by the time to complete the largest job. We needed a way to split them up.

One possibility was to find the slowest example groups and split them into multiple files. This is a game of whack-a-mole though: as we add more nodes, more and more files will need to be split up. It also creates artificial divisions in the spec files — does this new example go in `foo1_spec.rb` or `foo2_spec.rb`?

Another alternative is to treat each individual example as a job. However we found the communication overhead of farming out all the examples one by one was too high.

Instead we decided to slice up the example groups by the number of examples. For every 25 examples we created a slice. Then, instead of each item in the queue being an example group, each item was an Array of `<example group>`, `<slice index>`, `<slice count>`. Here’s an example:

<pre>
queue = [
  [BarSpec, 0, 3],
  [BarSpec, 1, 3],
  [BarSpec, 2, 3],
  [FooSpec, 0, 4],
  [FooSpec, 1, 4],
  [FooSpec, 2, 4],
  [FooSpec, 3, 4],
]
</pre>

The worker that picks up the job will only run the specs in it’s slice from that example group.

# 2. Reduce startup time

As we add more workers the sequential portion of the test run comsumes a higher proportion of the overall runtime (see [Amdahl’s law](https://en.wikipedia.org/wiki/Amdahl%27s_law)).

So the next step is to optimise the sequential portion (which is mainly the start-up time).

## Conditionally run migrations

The first thing that takes time is setting up the test environment. By far the most expensive operation here was running the database migrations.

We created a custom rake task that would check for missing or unwanted migrations in the test database and only run the migrations if it found any.

## Load specs on demand

The other main part of the startup time was loading all the spec files. By default, test-queue loads all the specs and then forks the workers to run them. When you have 1400 spec files to load this can take a while.

To speed this up, we stopped loading the specs before forking. Each worker loads the spec on demand before running them.
Because we haven’t loaded the specs before building the queue, we could no longer use example groups as our jobs. Instead we used the spec file paths. We still needed a way to slice up the larger specs, so we used the file size as a measure instead and created one slice for every 15,000 bytes.

Now our queue looks something like this:

<pre>
queue = [
  ["spec/unit/bar_spec.rb", 0, 2],
  ["spec/unit/bar_spec.rb", 1, 2],
  ["spec/unit/foo_spec.rb", 0, 4],
  ["spec/unit/foo_spec.rb", 1, 4],
  ["spec/unit/foo_spec.rb", 2, 4],
  ["spec/unit/foo_spec.rb", 3, 4],
]
</pre>

# Conclusion

Having the test suite run in 4 minutes is a great boost to productivity. In addition to the raw time saved running the build, there are additional savings in queuing time. This means we can get more rapid feedback on our code changes and get them into production more quickly, both of which greatly increase the productivity of our growing team.
