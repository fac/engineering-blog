---
:title: "Ruby 2.5: yield_self"
:created_at: 2017-12-19 12:00:00.000000000 +00:00
:kind: article
:tags:
- Ruby
:author: james
:slug: ruby-2-5-yield-self
---
<% excerpt do %>
`yield_self` is coming to Ruby 2.5. What is this long requested feature, and how does it work?
<% end %>

Some features take a while to get into a Ruby release. As you can see from the [original request](https://bugs.ruby-lang.org/issues/6721) on Ruby’s Redmine issue tracker, `yield_self` has been brewing for 5 years. It has been waiting on a good name, and the Ruby team has settled on one.

But what is it? To understand it, perhaps it is best to look at some existing features.
Ruby blocks are one of the ways of passing around functions in Ruby. Every method parameter list has an implicit block argument, which can be used within the method. One of the ways of activating a block is to call yield. For example:

<pre>
class GiftGiver
  ...
  def deliver(gift, &additional_behaviour)
    raise NoGiftError if @gift_tracker[gift.name] == 0
    @gift_tracker[gift.name] -= 1
    yield
  end
end

gifter = GiftGiver.new(gift)

# Santa Claus
gifter.deliver(new_ruby_version) do |gift|
  puts “Ho ho ho, a holly Merry Christmas. Here’s your #{gift}”
end

# Ruby core team
gifter.deliver(new_ruby_version) do |gift|
  puts “メリークリスマス!”
end
</pre>

The `tap` method was introduced in Ruby 1.9, and allows you to inspect on and act on a chain of methods. You can see how it's used in [the documentation](https://ruby-doc.org/core-2.4.2/Object.html#method-i-tap).

<pre>
(1..10)                .tap {|x| puts "original: #{x.inspect}"}
  .to_a                .tap {|x| puts "array: #{x.inspect}"}
  .select {|x| x%2==0} .tap {|x| puts "evens: #{x.inspect}"}
  .map {|x| x*x}       .tap {|x| puts "squares: #{x.inspect}"}
“original: 1..10”
“array: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
“evens: [2, 4, 6, 8, 10]
“squares:  [1,4,9,16,25,36,49,64,81,100]”
=> [1,4,9,16,25,36,49,64,81,100]
</pre>

The advantage of `tap` is that it always returns the value of the method `tap` was called on. This means in the above chain, apart from the `puts` calls, the result is the same as doing:

<pre>
(1..10)
  .to_a
  .select {|x| x%2==0}
  .map {|x| x*x}
=> [1,4,9,16,25,36,49,64,81,100]
</pre>

So, where does `yield_self` come in? It works similarly to `tap`, but it returns the value of the block you execute each time. 

<pre>
4.tap { |num| num*num } #=> 4
4.yield_self { |num| num*num } #=> 16
</pre>

This lets you chain together methods that ordinarily wouldn’t be chainable. An example of this is using class methods:

<pre>
filename.yield_self {|filename| File.open(filename)}.
               yield_self {|file| file.read}.
               yield_self {|contents| YAML.load(contents)}.
               yield_self {|yaml| yaml.fetch[“spec_directory”]}
</pre>

To do this naively without `yield_self`, you can nest arguments:

<pre>
YAML.load(File.open(filename).read).fetch[“spec_directory”]
</pre>

So, the trade off here is between a sequence and nesting arguments. There may be some cases where having some sequence methods allows you to be either more expressive or more comprehensible.

You can see a good discussion of this over on [Michał Łomnicki’s blog](https://mlomnicki.com/yield-self-in-ruby-25/). We’re interested to see where this goes and how frequently it is used!

