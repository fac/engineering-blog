---
:title: CoffeeScript with jQuery sprinkles
:created_at: 2012-03-23 11:01:30.000000000 +00:00
:kind: article
:tags:
- Javascript
:author: jonathan
:slug: coffeescript-with-jquery-sprinkles
---
### This is part two of a two part intro to CoffeeScript.

-   [part
    one](/2012/01/30/coffeescript-two-sugars-no-bitter-aftertaste/)
-   part two

So my [last
article](/2012/01/30/coffeescript-two-sugars-no-bitter-aftertaste/) on
CoffeeScript certainly seemed to provoke some thought. Some of you even
found it useful, which is all sorts of awesome. If you haven't had a
look at that article, I'd advise doing that first, as this one builds on
it.

* * * * *

There is one more thing I’d like to touch on with our [CoffeeScript
example](/2012/01/30/coffeescript-two-sugars-no-bitter-aftertaste/).
We’re using an inline event handler to validate the form, which is ugly
and obtrusive. With something like
[jQuery](http://jquery.com/ "jQuery"), fixing this would be a cinch, but
CoffeScript doesn’t give us any nice, cross-browser tools to achieve
this.

So let’s just use jQuery in our CoffeeScript!

I’m going to use the current version from jQuery’s content delivery
network:

    <script src="http://code.jquery.com/jquery-1.7.2.min.js"></script>
    <script src="/script/form.js"></script>

but you can [download the latest
one](http://docs.jquery.com/Downloading_jQuery "Download jQuery") itself
if you wish.

Let’s also remove the inline submit handler from the form tag:

    <form id="contact_form">

Now, to wire it all up. If this was simply jQuery, we’d do something
like:

    $('#contact_form').submit(function(){
      return validate(this);
    });

Look at all those brackets and braces! In CoffeeScript, this becomes
simply:

    $('#contact_form').submit ->
      validate this

Let’s step through this:

1.  strip out brackets and semi-colons:

        $('#contact_form').submit function()
          return validate this

2.  remove explicit returns:

        $('#contact_form').submit function()
          validate this

3.  replace `function()` with `->`:

        $('#contact_form').submit ->
          validate this

And as a final simplification, we can drop this to one line:

    $('#contact_form').submit -> validate this

Which reads well, and conveys its intent perfectly: “When submitted,
validate this”. One thing that may be annoying you is the brackets
around `'#contact_form'`. Why can’t we lose those? Consider:

    $ '#contact_form'.submit -> validate this

This will compile out to:

    $( '#contact_form'.submit(function(){ return validate(this); }) );

In other words, the *whole line* will be taken as the argument to the
`$` function call. This is actually a precedence issue, and the
idiomatic CoffeeScript way to resolve this is to lose the brackets
around the argument, but add them in around the operation you want to
take precendence. In other words:

    ($ '#contact_form').submit -> validate this

What we’re saying here is “call `submit` on the result of
`$ '#contact_form'`”. This looks odd if you’re used to JavaScript, but
is more in-keeping with the use of brackets in CoffeeScript to denote
precedence, not to pass arguments.

We’re nearly there. Lobbing this into our `form.coffee` file right now
won’t quite work, as we need to wait until the document is ready before
applying it. jQuery gives us that mechanism idiomatically:

    $(function(){
      // do things on page load
    });

or, after applying our CoffeeScript transliteration:

    $ -> // do things on page load

which gives us, in this case:

    $ -> ($ '#contact_form').submit -> validate this

Add this as the first line of `form.coffee` and we’re in business.

Finally, since we’re no longer calling the `validate` function from
outside the `form.coffee` file, we can switch from attaching the
function to the `window` object, keeping everything nice and
encapsulated:

    $ -> ($ '#contact_form').submit -> validate this

    validate = (form) ->
      errors = get_errors form, ['name','email']
      report errors
      errors.length == 0

    get_errors = (form,field_names) ->
      errors = []
      fields = (form.elements[name] for name in field_names)
      field.name for field in fields if field.value == ''

    report = (errors) ->
      alert "The form has errors:\n\n- " + errors.join("\n- ") if errors.length > 0

Fifteen lines of fresh, readable Coffee or forty lines of JavaScript?
Now *that* is a wakeup call.
