---
:title: A brief history of mobile
:created_at: 2016-03-11 16:00:00.000000000 +00:00
:kind: article
:tags:
- Mobile
- JavaScript
:author: ioan
:slug: a-brief-history-of-mobile
---

<% excerpt do %>
It's been a year since we launched our mobile app! In March 2015 we launched [FreeAgent Mobile for iOS](https://itunes.apple.com/app/freeagent-mobile-invoicing/id975591071?ls=1&mt=8). Initially limited to a small feature set, we quickly added new things over the next couple of months: expenses and mileage rebilling, updating and explaining bank transactions, and linking expenses and transactions to projects. In January 2016 we launched our [Android version](https://play.google.com/store/apps/details?id=com.freeagent.mobile) (with the same feature set as iOS), and more recently, we added the ability to upload files from any app on your phone, including Dropbox, Google Drive, and iCloud.

For FreeAgent Mobile's first birthday, I thought it would be good to show you what exactly makes it tick. Put on your seatbelt, 'cause this is gonna be a wild ride!
<% end %>

# Table of contents

- [History](#history)
  - [First try](#first-try)
  - [Second time's the charm?](#second-times-the-charm)
- [Initial explorations](#initial-explorations)
- [UI: React](#ui-react)
- [Data: Backbone](#data-backbone)
- [App state: Flux](#app-state-flux)
  - [Stores, Actions and Models](#stores-actions-and-models)
- [Tying it all together](#tying-it-all-together)
  - [UI structure](#ui-structure)
  - [Container vs. presentation components](#container-vs-presentation-components)
  - [Putting a page on screen](#putting-a-page-on-screen)
- [Testing](#testing)
  - [Unit testing](#unit-testing)
  - [Integration testing](#integration-testing)
- [Cordova](#cordova)
- [Conclusion](#conclusion)

# History

{:#history}

A few years ago, some of our engineers and designers were starting to become frustrated by the lack of a mobile offering. Phone apps were becoming increasingly important and, for many people, the primary way of interacting with their favourite services.

### First try

{:#first-try}

![FreeAgent Mobile web](/assets/images/2016/03-a-brief-history-of-mobile/fieldagent-v1.png){:.aligncenter}

During our [2012 Hack Week](http://engineering.freeagent.com/2012/08/20/hack-week-2-0-round-up) the team decided to implement mobile views inside our desktop app. These were rendered by our Rails back end and served as normal web pages to the user's browser. There was no app to install, and it was compatible with all mobile phones with a modern browser. At the time we were pretty happy with our mobile web solution, but we soon realised there was a demand for a separate app:

* People were searching for us in App Store and Play Store, but could only find third party apps;
* Loading a website to use an app was uncommon. Even though users could pin freeagent.com to their home screen, it still felt out of place alongside their other native apps;
* The mobile code was heavily dependent on desktop code, so with any change, we had to make sure we did not break mobile;
* The UI was built using vanilla JavaScript and jQuery. Hardly ideal for a dynamic mobile app.

### Second time's the charm?

{:#second-times-the-charm}

![FreeAgent Mobile app](/assets/images/2016/03-a-brief-history-of-mobile/fieldagent-v2.png){:.aligncenter}

In 2014, we got serious about putting an app in the stores. To remain focused, we decided to scope the initial version of the app to a small set of features: invoicing, expenses, and contacts.

Once we decided on the feature set, the focus turned to how we could technically achieve this. Based on our experience with the previous solution, we decided to keep this app completely separate. We knew we could make use of our existing API to fetch the data, and we could add any missing features to the API as and when they were needed by the mobile app. By "eating our own dog food", both the API and the mobile app could grow together into better tools for accessing FreeAgent.

The next question was: should this be a native or hybrid app? Here are some pros and cons:

* Native
  * \+ Quickly get access to new system APIs
  * \+ Use platform specific tools
  * \- Duplication of effort
  * \- Can't reuse any of our web infrastructure/tools
  * \- Updates need to go through review; we would have to plan sprints around it

* Hybrid
  * \+ One single codebase for both versions
  * \+ Can update app server-side to fix issues
  * \+ Allows us to use our existing web knowledge
  * \- Harder to implement platform-specific features (e.g. 3D touch, watch apps)
  * \- Still need to write native code for both iOS and Android

In the end we decided to go for a hybrid approach, as it seemed better suited to our team and the kind of app we were building.

# Initial explorations

{:#initial-explorations}

As our desktop app is built using [Rails](http://rubyonrails.org), this is what we tried to start with. Similar to our 2012 experiment, we tried having a Rails app generate the views, and render them in a native WebView container. However, just like before, this didn't feel very native or fluid enough for us. (Basecamp do use a similar solution [very](https://signalvnoise.com/posts/3766-hybrid-how-we-took-basecamp-multi-platform-with-a-tiny-team) [successfully](https://signalvnoise.com/posts/3743-hybrid-sweet-spot-native-navigation-web-content)).

# UI: React

{:#ui-react}

We then tried moving rendering of data to the client. At the time there were a few good candidates to create very interactive client side applications. We evaluated a few of them: Ember.js, Angular, Polymer, Backbone with Handlebars and [React](http://reactjs.org). At the time React did not have the huge community and support it has now, but it felt the best solution overall. It was built by Facebook and it looked interesting enough for us to give it a shot. Moreover, it was used in their main production apps, so we had some certainty that it will be maintained for a while and that it was suitable enough for a mobile UI. It also had a few advantages over other frameworks:

* **It was component-oriented**. This meant that we did not have to worry about raw DOM elements as much, and instead we could focus on the roles each part of our UI had, then compose these elements together.
* **Simple data flow**. By following a one-way data flow, and passing down data to children, it was really easy to understand what each component does when looking at its code. No more tangled data access mess, or forgetting to update DOM elements in callbacks!
* **Logic and presentation in same file**. No surprise here: being used to the HTML/CSS/JS separation of concerns, we thought having "HTML" inside JavaScript was a bit odd. But thinking about it, it became clear that React's way of putting logic and presentation together actually makes sense. After all, a component _is_ its logic and presentation. Why separate these, when they're so intertwined?
* **Virtual DOM**. Each framework we tried implemented some of the above features, but React was the only one that scored well in all of them, and the only one implementing a virtual DOM. Allowing the framework to optimise rendering for us was a great feature, as we did not have to worry about writing this logic ourselves.

# Data: Backbone

{:#data-backbone}

Once we picked the UI framework, we had to figure what to use for data fetching. Since it was quite simple, but powerful and popular, [Backbone](http://backbonejs.org) was our choice. It included everything we needed to map our API resources into JavaScript models and collections, and to communicate back and forth with the server. In order to use this with React, we made use of the [react.backbone](https://github.com/clayallsopp/react.backbone) mixin.

Besides the vanilla Backbone behaviour we also added code for allowing Rails-style relationships between models. For example, we can say that an `Invoice.hasMany('invoice_items', InvoiceItem)`, and our Invoice model now knows where to look and how to parse these invoice items into a Backbone collection of `InvoiceItem` models. Or we can say that an `Invoice.hasOne('contact', Contact)`. This proved quite useful, as we could just call `invoice.invoiceItems()` or `invoice.contact()` from our React views without having to worry about the underlying logic needed to fetch these associations. Moreover, if the association wasn't yet loaded, it would be grabbed from the API when calling these methods.

We extracted this logic, along with the models and collections, into a JavaScript library which we hope to open-source in the future. There is still some work to do before we can do that, though (see below).

While it has served us well over the past 18 months, soon we will replace Backbone with [Immutable.js](https://facebook.github.io/immutable-js). This change should make it easier for us to optimise the mobile app and reduce the number of bugs that can arise from self-mutating objects.

# App state: Flux

{:#app-state-flux}

Flux is an application architecture that promotes unidirectional data flow and global app state. This fits React nicely as they are both built on the same ideas. [Here](https://facebook.github.io/flux/docs/overview.html#structure-and-data-flow)  you can see a diagram of the concepts involved in the Flux architecture.

While our mobile app worked fine without Flux for the first few months, once we started writing code for banking we realised we needed a better architecture. One of the workflow requirements was: once a transaction is saved, the user should be returned to the bank account screen to act on their other transactions. The transaction would be saved in the background with its item disabled in the transactions list, only to be enabled again once the save was successful.

![Banking workflow](/assets/images/2016/03-a-brief-history-of-mobile/banking.gif){:.aligncenter}

This requirement prompted us to re-think how we structure data access in our app. Before introducing Flux, we were simply setting up the needed Backbone models when reaching a certain URL, passing them into the page component. The page component would in turn pass these down to its child views. This worked fine for isolated pages which didn't have to "communicate" with each other, but this communication is exactly what our workflow required.

We realised we needed some sort of property or event to tell the bank account page hosting the transactions list that _this transaction is saving, disable it_ and _this transaction has saved, enable it again_. Because we could only send events and properties to the visible page on the screen, our solution had to involve some sort of global state.

We tried implementing the workflow without Flux, but our solution ended up looking very much like Flux, so we decided to adopt one of the frameworks. We looked at [this](https://github.com/kriasoft/react-starter-kit/issues/22) nice list of libraries, and decided that [alt](http://alt.js.org) was a good fit for us. We liked the fact that under the hood it was really using Facebook's flux implementation, and its API was simple to use and understand.

### Stores, Actions, and Models

{:#stores-actions-and-models}

Our Flux implementation ended up looking like the following. For each model we have a store, and corresponding actions:

* Invoice <- InvoiceStore <- InvoiceActions
* BankAccount <- BankAccountStore <- BankAccountActions
* etc.

In stores, data is kept in a _master hash_ keyed by the model ID to allow fast lookup of any model. Even though we have pagination and filters, the master hash is flat. No matter which filter or page the model is loaded from, it gets added to the master hash. We also have a _filters_ hash which is keyed, as you might guess, by the filter name. Each filter is an object with the following properties:

* models: a `Set` of IDs pointing to the master hash
* page: number of currently loaded page
* hasMore: whether there are more pages available to load

Each store also contains a few read-only methods to grab data from it. For example, the `BankTransactionStore` has methods such as `getTransaction(id)`, `getTransactions(filter)`, and `hasMorePages(filter)`.

Apart from these methods, we have a few more which are registered to actions. As described above, each model has one or more "associated" actions. These actions, when called by views, manipulate data (loaded from API, or existing data in stores) and then dispatch an event to any store that may be interested in the event. For example, here's how we listen for events in `BankTransactionStore`:

<script src="https://gist.github.com/dragossh/89675a3397acb6eb24e9.js"></script>

When `TransactionExplanationActions` dispatches one of these events, the store is alerted and the associated method is called to change the data inside the store. When data inside the store changes, any views listening to the store are alerted and will render themselves again.

This architecture made it incredibly easy to implement our original workflow: just have the two views, bank account and transaction show, listen and fetch data from `BankTransactionStore`, add a `processing` attribute to the transactions in the store, and enable or disable the transaction in the list based on the value of `processing`.

# Tying it all together

{:#tying-it-all-together}

Now that I've explained the technologies behind the app, let's see how they all fit together to power the FreeAgent Mobile apps.

### UI structure

{:#ui-structure}

At the root of our UI, we have a `MobilePager` component. It is responsible for rendering and keeping track of visited pages (including the page visible on screen), and helping us push, pop, or replace pages. It also animates transitions between any pushes or pops.

Each page that is pushed to the mobile pager is bound to a Backbone object (generally via the `model` or `collection` property), and mostly render `NavBar` and `ViewContent` components. The pages are usually bound to a certain URL, for example `Settings` -> `/#settings`. This is how the actual code looks like:

<script src="https://gist.github.com/dragossh/f64a0673e6f3a8afef6c.js"></script>

As you can see, the view is pretty simple, rendering `NavBar` and `ViewContent` as child components. Most other views follow the same pattern, with varying levels of complexity.

### Container vs presentation components

{:#container-vs-presentation-components}

In order to show interesting stuff, each page has to render its own subcomponents. In the `ViewContent` of each page, we add these subcomponents, whether they are containers or simply presentation components.

What's a container and what's a presentation component, you ask? [Here](https://medium.com/@learnreact/container-components-c0e67432e005#.qwu9v4nmm) is a great article explaining the difference. In a nutshell, the idea is to make the components that show data as generic as possible, in order to promote reuse and testability. Then more complex logic or data fetching can be done in containers. These containers wrap around the presentation components, rendering them and providing them with the required data.

### Putting a page on screen

{:#putting-a-page-on-screen}

When the app boots, the first thing we do is to render the `MobilePager`:

<script src="https://gist.github.com/dragossh/d0e923499636724767aa.js"></script>

Now we can push any pages we want on the screen. Let's navigate `/#home`. The route code is triggered, setting up the needed models, fetching the data, and passing them down to `HomeView` component:

<script src="https://gist.github.com/dragossh/ad3cf31b3601522ca9e2.js"></script>

# Testing

{:#testing}

In order to test our app, we have two different stacks: one for unit testing and one for integration testing. Unit testing allows us to check our React components in isolation, while integration tests simulate a real-world workflow of the app and check that everything is working well together.

### Unit testing

{:#unit-testing}

We use [Jasmine](http://jasmine.github.io) for unit tests. Jasmine is a JavaScript testing framework, similar to  [Mocha](https://mochajs.org) + [Chai](http://chaijs.com),  [Karma](https://karma-runner.github.io), or [QUnit](http://qunitjs.com). It includes expectations, spies, and has the ability to mock objects and AJAX requests. We also make heavy use of React's `TestUtils` to simulate events, find DOM nodes based on a React type, or mock components.

Almost each presentation component we wrote has its own spec, asserting that it renders data and child components as we expect. The usual unit test contains these steps:

* Render component with some props
* Assert something about its output
* Assert that it has expected behaviour (e.g. calling event handlers)

### Integration testing

{:#integration-testing}

As I mentioned above, integration tests have a broader scope. Instead of checking the independent behaviour of a certain component, it checks that everything works together and makes sure the right API calls are being made. This can catch subtle bugs that are not uncovered by unit tests.

Thanks to FreeAgent Mobile being hosted inside a Rails app during development, we can easily use [rspec](http://rspec.info) along with [capybara](http://jnicklas.github.io/capybara) to write these integration tests. We have a couple of specs, each related to the section it checks e.g. `invoice_workflow_spec.rb`, `manage_contacts_spec.rb`,  `bank_accounts_spec.rb`. Each of these specs do the same thing:

* Login into the app
* Browse through the pages in that section
* Tap controls, modify some data
* Check API calls are made
* Check that the correct pages are being displayed

# Cordova

{:#cordova}

There is one last piece of the puzzle I haven't mentioned yet, and that is [Cordova](https://cordova.apache.org). This tool allows us to get our JavaScript, HTML, and CSS files, and package them into native iOS and Android apps. It also offers us a way to call native code from our JavaScript, for features such as camera, status bar management, and native file pickers.

# Conclusion

{:#conclusion}

I hope this tour of our mobile tech stack was interesting and helpful! We're having a lot of fun building the mobile app with these technologies, and we're really happy with the feedback we've received so far. Look forward to more stuff coming out of the mobile team soon. If you have any questions or feedback, please leave a comment and we'll get back to you.