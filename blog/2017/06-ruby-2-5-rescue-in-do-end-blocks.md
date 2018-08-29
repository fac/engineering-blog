---
:title: "Ruby 2.5: Not Blocking My Rescue"
:created_at: 2017-12-20 12:00:00.000000000 +00:00
:kind: article
:tags:
- Ruby
:author: james
:slug: ruby-2-5-not-blocking-my-rescue
---
<% excerpt do %>
Rescuing specific exceptions excessively can cause problems, but if you've ever had need to rescue within a `do/end` block, you might have found yourself using wordy syntax. Ruby 2.5 has a solution for you.
<% end %>

In Ruby 2.5, we’ll get a little syntactic sugar for handling exceptions inside `do/end` blocks. You can see the [feature discussion](https://bugs.ruby-lang.org/issues/12906) on Ruby’s Redmine instance. If you’ve ever used the shorthand for rescuing inside a method without using `begin`/`end` keywords, this is basically that but inside blocks.

Below we’re going to work through a bit of code, but we’re not going to define all the methods.

Imagine a `Santa` class, and an algorithm for Santa Claus arriving in town. Santa needs some paper to jot down delivery details. If there is no paper, we raise an error.

<pre>
class Santa
  def initialize(good, bad)
    @nice_children = good
    @naughty_children = bad
  end

  def make_list
    raise NoPaperError if @paper.nil?
    @list = nice_children.gifts.map(&:details)
  end

  def check_list
    @list.verify_children(nice_children, naughty_children)
  end

   # other Santastic behaviour below
end
</pre>
We’ll define a global `prepare` method that takes a block. In Ruby 2.4 when we send the `make_list` message to Santa within the block, we need to use a full `begin/rescue/end` clause to describe the behaviour:

<pre>
def song(children)
  santa = Santa.new(children.good, children.judged_capriciously_by_society)
  prepare do
    # must use `begin` here in Ruby 2.4 and earlier
    begin
      santa.make_list
      2.times { santa.check_list }
    rescue NoPaperError
       santa.request_paper
    ensure
       music_stops
    end
  end
end

def prepare(&block)
  yield
end
</pre>

If we try to use the shorthand common in `Class`es, we hit an exception:

<pre>
def song(children)
  santa = Santa.new(children.good, children.judged_capriciously_by_society)
  prepare do
    santa.make_list
    2.times { santa.check_list }

    rescue NoPaperError
      santa.request_paper
    ensure
      music_stops
    end
  end
end

song(children) #=> SyntaxError:  syntax error, unexpected keyword_rescue, expecting keyword_end
</pre>

In Ruby 2.5 though, we can do this handily:
<pre>
def song(children)
  santa = Santa.new(children.good, children.judged_capriciously_by_society)
  prepare do
    # no `begin` keyword!
    santa.make_list
    2.times { santa.check_list }

    rescue NoPaperError
      santa.request_paper
    ensure
      music_stops
    end
  end
end

song(children) # => “music stops”
</pre>

So that’s nice. See you tomorrow for more Ruby 2.5 news!

