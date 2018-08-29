---
:title: ! 'CoffeeScript: two sugars, no bitter aftertaste'
:created_at: 2012-01-30 09:53:57.000000000 +00:00
:kind: article
:tags:
- Javascript
- Ruby on Rails
:author: jonathan
:slug: coffeescript-two-sugars-no-bitter-aftertaste
---
### This is part one of a two part intro to CoffeeScript.

-   part one
-   [part two](/2012/03/23/coffeescript-with-jquery-sprinkles/)

* * * * *

The FreeAgent web application runs on Rails, and around the corner for
us is an upgrade to Rails 3.1. This will bring many benefits to
performance, but one of the things I'm most excited about is the asset
pipeline. This makes JS and CSS assets first-class Rails citizens, and
as a bonus, allows us built-in access to pre-compilers like CoffeeScript
and Sass.

I've been cutting JavaScript for as long as I've been coding
for the web, so CoffeeScript has me all excited.

Waiter! There's some soup in my coffee!
---------------------------------------

JavaScript isn’t very readable, and unreadable code is hard to maintain.
Compared with Ruby or Python, there are brackets, braces and quotes
everywhere. Often, there’s more syntactical soup than software.

[CoffeeScript](http://jashkenas.github.com/coffee-script/ "CoffeeScript")
isn’t a framework, but instead compiles to runnable JavaScript. You
write CoffeeScript, compile it, and out pops clean, tight JavaScript
ready for the browser. You get optimised JavaScript, but work with
clean, understandable, maintainable code. 

CoffeeScript starts to make
real sense once you’ve written some, so we’ll get to that as fast as we
can. First, let’s look at installing the CoffeeScript compiler, so we
can have it convert our CoffeeScript files into JavaScript that we can
load in our browser. 

To get CoffeeScript installed on your development
machine, you’ll need a \*nix-like environment, a text editor, a
terminal, and a browser to check the results. First we install
[node.js](http://nodejs.org/ "Node.js") (a JavaScript runtime that
CoffeeScript needs to do its magic). We then install node’s package
manager (npm) and use that to install CoffeeScript itself. On Mac OS X,
the simplest way to do this is with Homebrew. Make sure you have XCode
installed, then follow the instructions to install Homebrew, or just
open a terminal session and type:

    /usr/bin/ruby -e "$(curl -fsSL https://raw.github.com/gist/323731)"

Then use Homebrew to install node.js:

    brew install node

Finally, install [npm](http://npmjs.org/ "npm") (a package manager for
node), and use that to install CoffeeScript:

    curl http://npmjs.org/install.sh | sh
    npm install -g coffee-script

If you’re running Linux, your package manager can install node, then
install npm and CoffeeScript. If, on the other hand, you’re using
Windows, try following Matthew Podwysocki’s instructions at
[CodeBetter](http://codebetter.com/matthewpodwysocki/2010/09/08/getting-started-with-node-js-on-windows/ "Getting Started with node.js on Windows").

Okay, so now you’re up and running with CoffeeScript. Let’s dive in.

CoffeeScript, you validate me
-----------------------------

What I want to run through here is the CoffeeScript syntax, and how
expressive it can be. I could build up a really complicated little app,
and that would be fun, but it would also be, well, complicated.

Let’s
keep things simple but practical so we can focus on CoffeeScript, not
the problem domain. Let’s validate a form!

    <form id="contact_form">
      <ol>
        <li> 
          <label>Name</label>
          <input type="text" name="name">
        </li>
        <li>
          <label>Email</label>
          <input type="email" name="email">
        </li>
        <li>
          <label>Enquiry</label>
          <textarea name="enquiry"></textarea>
        </li>
      </ol>
      <ol>
        <li><input type="submit"></li>
      </ol>
    </form>

Woo! Exciting! Let’s crack open `form.coffee` and see what mischief we
can get up to.

    required_field_names = ['name', 'email']

So far, so boring. That’s just JavaScript! True, but let’s drop to the
shell and compile that file out to JavaScript:

    coffee -c form.coffee

gives us `form.js`, which looks like this:

    (function(){
      var required_field_names;
      required_field_names = ['name','email];
    }).call(this);

Woah.

What’s happened? The `required_field_names` variable has been
scoped by `var`, and the whole script has been wrapped in a namespace.
This protects us against one of the most common sources of bugs in
JavaScript: accidental global variables. In CoffeeScript, variables are
local by default, instead of global as in regular JavaScript. If you’ve
never worried about scoping in JavaScript before, you’re very lucky. If
you have, then this is a lifesaver.

The first sip
-------------

Let’s include the js file in our form:

    <head>
      <script src="form.js"></script>
    </head>

And then, horribly, let’s add an event handler to the form. Remember,
CoffeeScript isn’t a framework:

    <form onsubmit="return validate(this);">

Let’s declare our required fields array within this `validate` function,
and `return false` to prevent the form from submitting during
development. In plain JS, we might write the following:

    var validate = function( form ) {
      var required_fields_names = ['name', 'email'];
      return false;
    }

Now, CoffeeScript handles the variable declaration, so we can lose
`var`. Additionally, function declarations use a more concise notation,
so

    function( args ) { ... some code ... }

becomes

    ( args ) -> ... some code ...

and instead of braces, we use indentation to describe blocks. So we get:

    validate = (form) ->
      required_field_names = ['name', 'email']
      return false

We’ve also lost the semi-colons at the end of each line. Finally, like
Ruby, the result of the last statement executed in the function is
automatically returned. So we can lose the explicit return:

    validate = (form) ->
      required_field_names = ['name', 'email']
      false

Compile it out with `coffee -c form.coffee`, check `form.js`, reload the
form in your browser and… the form submits. What’s gone wrong?

CoffeeScript creates everything in a namespace with a local scope. That
means our `validate` function is so far only available to be called
within the scope of the `form.js` file, and not outside. This prevents
another function called `validate` from clobbering our definition, but
isn’t very helpful. To get around this, CoffeeScript makes us explicitly
declare variables we wish to have a global scope:

    window.validate = (form) -> …

is what we need. Compile again, reload the form, submit and… no
submission. Progress! 

If you’re anything like me, typing
`coffee -c form.coffee` each time you make a change is starting to get
annoying. If you’re so inclined, you could use something like
[CodeKit](http://incident57.com/codekit/ "Code Kit"), but me, I like my
command line. Luckily, the `coffee` command has a `watch` option.
Running:

    coffee -cw *.coffee

will launch the compiler and leave it running, recompiling any file that
matches the pattern as it changes. Nice.

Grinding the beans
------------------

We have a list of required field names, and a submit handler to check
them. Let’s get stuck into grabbing the fields themselves. Again, in
JavaScript, you might do something like:

    var required_fields = [];
    for ( var name in required_field_names ) {
      var field = form.elements[name];
      required_fields.push( field );
    }

to build up an array of actual fields to check. Yes, jQuery would make
this simpler, but we’ll see about that later. Transliterating into
CoffeScript, as a first pass, we might write:

    required_fields = []
    for name in required_field_names
      field = form.elements[name]
      required_fields.push( field )

Not a huge saving, but we can go one better with the `for` loop:

    required_fields = for name in required_field_names
      form.elements[name]

In other words, `for` will act as a `map`, collecting together the
returns of each iteration and returing those in an array. Finally,
CoffeeScript gives us a little bit of magic to turn this into a one
liner:

    required_fields = (form.elements[name] for name in required_field_names)

This reads like a sentence:

> Required Fields are the form elements for each Required Field Name

But wait. Parentheses? I thought this was CoffeeScript! We don’t need no
stinking parentheses!

Well, turns out we do. Parentheses in CoffeeScript
are still allowable. In fact, they are used primarily to ensure
precedence (just as they are in JavaScript), or sometimes simply to
increase readability. 

We’ve distilled five lines of JavaScript down to a
single, clean line of code that expresses precisely what it does.
`validate` itself is now four lines long. The compiled JavaScript is
sitting at around 18 lines of bullet-proof, tight and memory efficient
code. 

There’s a lot to like about CoffeeScript.

Checking the roast
------------------

We have an array of input elements, so let’s check that each has a
value. In JavaScript, we could do this:

    var errors = [];
    for ( var field in required_fields ) {
      if ( field.value == '' ) {
        errors.push( field.name );
      }
    }

This would correct an array of bad field names. Let’s CoffeeScript this
up:

    errors = []
    for field in required_fields
      if field.value == ''
        errors.push field_name

This doesn’t seem much of a saving. Notice, however, that there’s only a
single statement in the `if` block. This means we can use the same trick
we employed in the `for` block — putting the conditional statement in
front of the condition:

    errors = []
    for field in required_fields
      errors.push field_name if field.value == ''

Now there’s only a single line in the `for` block, so we could repeat
the trick and move everything on to a single line:

    errors = [] ( errors.push field_name if field.value == '' ) for field in required_fields

Note our parentheses again, to help clarify what’s happening to what.

So, our CoffeeScript now looks like this:

    window.validate = (form) ->
      required_field_names = ['name', 'email']
      errors = []
      required_fields = (form.elements[name] for name in required_field_names
      (errors.push field.name if field.value == '') for field in required_fields
      false

There are two things left to do: prevent the form from submitting only
if we have errors, and then report those errors back to the user.

Serving the perfect cup
-----------------------

Preventing submission on error is now trivial. Replace the last line of
the function with a check on the number of errors:

    window.validate = (form) ->
      ...
      errors.length == 0

`validate` now explicitly provides the “yes or now” answer to: are there
zero errors on the form? This is very readable and maintainable: the
final line of the function sums up perfectly what the function does. The
error reporting could be similarly simple, adding the following before
the return line:

    alert errors.join(',') if errors.length > 0

This is too simple for me, though: no descriptions, just a list of field
names. Let’s break this out into an error handling function — `report`.
Add this line instead of the `alert`:

    report errors

then add the following function below `validate`:

    report = (errors) ->
      alert "This form has errors:\n\n" + errors.join("\n- ") if errors.length > 0

Our final source looks like this:

    window.validate = (form) ->
      required_field_names = ['name', 'email']
      errors = []
      required_fields = (form.elements[name] for name in required_field_names)
      (errors.push field.name if field.value == '') for field in required_fields
      report errors
      errors.length == 0

    report = (errors) ->
      alert "This form has errors:\n\n" + errors.join("\n- ") if errors.length > 0

Since we’ve started extracting concerns, let’s go a step further.
Looking at the `validate` function, it does four things: defines fields
that should be completed; collects errors from those fields; reports
those errors, and returns false unless it has found no errors.

Cleaning up
-----------

This sort of quick extraction has always been a pain in JavaScript.
Creating functions can lead to scoping issues, and the syntactical soup
to ensure correct scope often prevents extraction from adding to the
readability of the code. With CoffeeScript, pulling out functionality
such as the error handling is trivial and does nothing but aid
readability. We could continue this to clean up validation further:

    window.validate = (form) ->
      errors = get_errors form, ['name', 'email']
      report errors
      errors.length == 0

where `get_errors` just extracts the error-scanning code:

    get_errors = (form, field_names) ->
      errors = []
      fields = ( form.elements[name] for name in field_names )
      field.name for field in fields when field.value == ''

we could get even more concise and make `report` return a value
dependent on whether there are any errors to report, which would let us
boil `validate` down to:

    window.validate = (form) ->
      report( get_errors form, ['name', 'email] )

This is probably a step to far for just now, but look what we’re doing:
we’re actually discussing how to make the code tidier and more readable
instead of simply trying to figure out what the heck the code is trying
to do in the first place. 

That, alone, is the biggest win CoffeeScript
gives you: You’re no longer tasked with first conquering your language
before you can tackle the domain problem.

**Updated:** the `get_errors`
function definition thanks to a suggestion by [Robin
Wellner](/2012/01/30/coffeescript-two-sugars-no-bitter-aftertaste/#comment-113).
