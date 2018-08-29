---
:title: "Ruby 2.5: How's That Backtrace looking?"
:created_at: 2017-12-21 12:00:00.000000000 +00:00
:kind: article
:tags:
- Ruby
:author: james
:slug: ruby-2-5-how-is-that-backtrace
---
It’s Christmas night, the air is cool and the stars are unseen through the heavy cloud. Children have left [tablet](https://en.wikipedia.org/wiki/Tablet_(confectionery)) and a nip of whisky out for Santa, and carrots out for the reindeer. One of the children wakes up! Rushing out of bed, she heads to the living room, curious to see if she can catch Santa and reindeer in the act of present delivery.

When she gets there, she finds the food and drink have gone. However there are footprints and reindeer tracks, which lead outside.

As developers, we’re fairly used to following a trail of clues from a point of interest. When a program you’re developing crashes or raises a controlled exception, your tools will typically provide you with as much information as it can about the error. In Ruby, this typically includes a [stack trace](https://en.wikipedia.org/wiki/Stack_trace), showing all the call points on the way to the error. Ruby calls this a “backtrace”.

<pre>
def turtle_doves
   partridge_in_pear_trees = 1
   partridges(partridge_in_pear_trees)
end

def partridges(number)
  raise “Shouldn’t this be recursive?”
end

turtle_doves

$ ruby test.rb
test.rb:7:in `partridges': Shouldn't this be recursive? (RuntimeError)
	from test.rb:3:in `turtle_doves'
	from test.rb:10:in `&lt;main&gt;'
</pre>

Something you may not have thought about much is the order in which this stack trace is presented to you. As you can see above, in Ruby 2.4 and earlier you’ll see the error, followed by the location of the code that caused the error, and then site that called that location and so on back to the first line of the program. On a command line, this means that you’ll often see the least useful thing closest to your cursor.

For small stack traces, this is fairly handy! It puts the error information right next to the line that caused the error. For longer stack traces, that you’d typically see in game programming or Rack based web applications, it means you need to scroll off screen to find the error site.

Four years ago, someone [suggested a Ruby feature](https://bugs.ruby-lang.org/issues/8661
) to allow the order of this stack trace to be reversed. This feature will land in Ruby 2.5!

This is backtrace we get from running the same function above under 2.5:

<pre>
$ ruby test.rb
Traceback (most recent call last):
  2: from test.rb:10:in `&lt;main&gt;'
  1: from test.rb:3:in `turtle_doves'
test.rb:7:in `partridges': Shouldn't this be recursive? (RuntimeError)
</pre>

This is really handy for those longer stack traces, and is actually how other languages (such as Python) have handled stack traces for a while. However, if you have tooling that parses error output like this (such as a test tool), it will likely be broken by this change. The core team decided to make this a non-optional change for this reason; it means that tooling like this doesn’t have to work out which ordering is being used.

I’m really looking forward to being able to use this; I suspect it will make debugging much easier.

See you tomorrow for one more post about a Ruby 2.5 change!
