---
:title: React Europe 2015
:created_at: 2015-07-31 16:00:00.000000000 +00:00
:kind: article
:tags:
- conferences
:author: ioan
:slug: react-europe-2015
---

We love [React](http://facebook.github.io/react) here at FreeAgent. Our team has been using it heavily to build the mobile app, with very few issues. Because it’s so easy to use and helps with maintenance of complex UIs, we’ve also started migrating parts of the desktop app to it, such as inline bank explanations.

At the beginning of July, a few of us went to Paris to attend the first [React Europe](https://www.react-europe.org) conference. Here are some of the topics we felt were interesting, and important to us going forward:

# ES6 is the future!

Because FreeAgent is built in Rails, our language of choice for client-side is [CoffeeScript](http://coffeescript.org). It ships with Rails by default, and allows us to write JavaScript in a similar style to Ruby. We use CoffeeScript everywhere – in FreeAgent, in our JavaScript API library, and in our mobile project. However, not a lot of projects outside of the Ruby community make use of it. At the conference and in React libraries most of the code we saw was written using ES6.

ES6 includes a syntax similar to CoffeeScript and borrows many of its features. Some of the enhancements in ES6 are: lambdas, classes, iterators, reflections, much nicer module syntax, native promises, and being able to mark functions as async (which avoids callback hell and does not require any third party libraries).

While browsers have [started implementing](http://kangax.github.io/compat-table/es6) ES6 features, some are not yet supported. Older browsers are also not able to take advantage of the new syntax. For this reason, [Babel](http://babeljs.io) appeared, which allows developers to write their source code in ES6, and output as something else (likely ES5). As more and more libraries take advantage of it, ES6 and Babel look like a promising alternative to CoffeeScript.

# GraphQL

Building APIs is easy. Maintaining them is hard. Keeping compatibility with various apps and multiple versions is even harder. For this reason, Facebook have been building a technology called GraphQL in order to solve all these issues. They’ve been successful, and for the past 3 years their APIs have been powered by it. At React Europe they’ve announced their desire to make it a public specification and allow developers to implement GraphQL in their projects.

So what is GraphQL? It’s easier to explain what it isn’t: it’s not a storage engine, and it is language and data source agnostic. This means it should fit in nicely with any architecture, including Rails projects.

The way GraphQL works is as follows: the structure of the data is defined as various types, then these types are combined to get the desired API. The type definitions are similar to JSON and they map to the actual response clients see. One nice advantage of this type system: it exposes the type metadata to clients, so they can see exactly what fields are available and request only the ones they need, in order to reduce response size. Another interesting result is that Facebook are able to support all previous versions of their apps without breaking them, going back at least 3 years.

Because their internal implementation is tied very much to how Facebook works, they are releasing a [draft](http://facebook.github.io/graphql) spec along with a JavaScript based server (this will be released in August). There is more work to be done here, but it looks very good. No promises, but we are excited to play around with this in order to enhance our API.

# Flux

As with React, Flux came out of Facebook’s desire to have an easier way to manage the UIs of their applications. In this case, the _state_ of their UIs.

The idea of Flux is that not all app state is local, and some state is shared across components (for example, the currently logged in user, or company settings). With Flux, there are stores in which this global state is kept, and these stores are the single source of truth for all components. As you can imagine, this makes it easier to reason about the state of the system than randomly accessing and setting data from components.

Because the concept is so simple but adaptable, there are multiple of implementations of the concept, including [Fluxible](https://github.com/yahoo/fluxible), [Reflux](https://github.com/reflux/refluxjs), [Barracks](https://github.com/yoshuawuyts/barracks), and [McFly](https://github.com/kenwheeler/mcfly).

But the most amazing one, and the one demoed at the conference, was [Redux](https://github.com/gaearon/redux). It incorporates concepts from functional programming and embraces immutability and atomicity. The result: an amazing demo on hot reloading and stepping back into past state.

# Relay

It’s no surprise that declarative approaches lead to fewer bugs than imperative ones. This has been proven right for UIs by React, and Relay aims to do the same for data communication. Instead of AJAX calls inside (or outside) React components, a component declaratively specifies the data it needs. Then Relay automagically fetches and stores the data, without having to do AJAX calls all over the codebase.

Combined with GraphQL, Relay seems like a really nice alternative to the old data fetching methods.

There are a few libraries available, such as [react-transmit](https://github.com/RickWong/react-transmit) and [turbine](https://github.com/chute/turbine).

# Hybrid apps

This is a hot topic amongst developers. Some claim you need to go native to get integrated apps. Others say this can be done very well with hybrid apps. We certainly ran into some issues while developing our [iOS app](http://www.freeagent.com/central/introducing-freeagent-mobile-ios), but after what we saw at the conference, we’re convinced hybrid can work well, and that these issues can be overcome.

[TouchstoneJS](http://touchstonejs.io) was introduced as a React library to build performant, native-looking mobile UIs. The talk was very impressive, and everybody was surprised when it was revealed that the [official React Europe app](http://thinkmill.com.au/react-europe) was built using Touchstone! We’ve all been using it during the conference, thinking it was completely native.

Here are some takeaways from the talk:

1. Use React! :)
2. Touch
- Users expect instant feedback, anything less will feel “wrong”
- Scrolling should cancel taps
- Drag away from a button should stop highlighting it. When returning to it, the highlight should be applied
- These are quite hard to get right, but luckily there’s a library called [react-tappable](https://github.com/JedWatson/react-tappable) which implements all this
3. Layout
- Use flex box, as it the most natural way to lay out complex mobile UIs
- Make table headers sticky using `-webkit-sticky` attribute
- When scrolling past beginning / end, pages should use rubber banding: this can be achieved with some CSS tricks, which add and subtract some padding / margins from top and bottom.
- Keep scroll position of previous page
- Another library to help with all this: [react-container](https://github.com/JedWatson/react-container)

# Conclusion

This was one of the best conferences we’ve been to. There were many great talks, so we recommend watching them on YouTube: [day 1](https://www.youtube.com/playlist?list=PLCC436JpVnK0Phxld2dD4tM4xPMxJCiRD) and [day 2](https://www.youtube.com/playlist?list=PLCC436JpVnK3HvUSAHpt-LRJkIK8pQG6R).

If there is one thing to take away from React Europe, is that the React stack changes rapidly, and developers should be prepared to change with it. A lot of these technologies did not exist a year ago, but they offer clear advantages over classical approaches. Many companies, including Facebook, Twitter, and Netflix, already adopted them and use them in production systems, so the level of polish is already pretty high. If you didn’t yet make the jump to React… we suggest you give it a try, it may surprise you!
