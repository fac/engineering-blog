---
:title: Schrodinger's Ruby array
:created_at: 2016-11-08 14:00:00.000000000 +00:00
:kind: article
:tags:
- ruby
:author: caius
---

<% excerpt do %>
When can a single array instance in Ruby both be empty and contain items simultaneously? Read on, and find out!
<% end %>

We have an application that receives messages from an external source, places them into an inbox (array) for a consumer to handle the next time it wakes up. All within a single Ruby process.

We were running a copy of the application on a new server for QA purposes (with identical code to elsewhere), and finding that the events weren't being handled from the inbox, which was odd in itself. The *super* strange revelation came when we started trying to debug what was happening however.

In the consumer, we added logging to check how many items were in the inbox when it woke up and checked. At the same time, in the producer, we added logging to say how many items there were in the inbox when it added another item to it.

This gave us the following logs:


    [producer] Received event #1; placed in inbox: 1 items
    [consumer] Woken up; inbox: 0 items

    [producer] Received event #2; placed in inbox: 2 items
    [consumer] Woken up; inbox: 0 items

Okay, so maybe the producer and consumer aren't using the same inbox object (an `Array` instance in this case.) We added more debug logging:

    [producer] Received event #1; placed in inbox: 1 items, object id #123
    [consumer] Woken up; inbox: 0 items, object id #123

    [producer] Received event #2; placed in inbox: 2 items, object id #123
    [consumer] Woken up; inbox: 0 items, object id #123

Wait what now? The producer says the inbox has 2 items, but the consumer (looking at the **same** Ruby object - ids are unique within current process) sees no items in that inbox?!

This continued to baffle us for a while (there was a handful of us on a video chat debugging this), until someone wondered if forking child processes had anything to do with it—The application runs under Unicorn, a web server which forks child worker processes from an initial master process.

We then added the process id to the debug lines and suddenly everything made sense:

    [producer pid: 87] Received event #1; placed in inbox: 1 items, object id #123
    [consumer pid: 88] Woken up; inbox: 0 items, object id #123

    [producer pid: 87] Received event #2; placed in inbox: 2 items, object id #123
    [consumer pid: 88] Woken up; inbox: 0 items, object id #123

The producer was running in the master Ruby process and the consumer thread was running in the child process. The inbox object was initialized in the master process **before** the fork however, so had the same object id in both processes.

So the answer is, an array with identical object id can be both empty and contain items when it's in two separate processes. And the easiest way to run into it is with things created before a process forks!

\[If you like solving problems like this and you're looking for a new career challenge, we'd love to hear from you! <https://www.freeagent.com/company/careers>\]
