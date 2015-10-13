---
title:  The Future of Web Development - React, Falcor, and ES6
author: Ray Nicholus
categories: react falcor webpack web server JSON HTTP node.js JavaScript ES6
excerpt: "The future of web application development looks a bit different than what we are all used to. I'll show you how to build a simple full-stack JavaScript app using Node.js on the backend, React on the frontend, Webpack for client-side module support, and Netflix's Falcor as an efficient and intuitive alternative to the traditional REST API."
---

In this article, I'm going provide a glimpse into the future of web development. You will gain a new perspective on structuring a user interface, server, and data endpoints. In other words, I will cover the full "stack" - both the browser and server code - and you will be able to examine and execute all of the referenced code in [a fully-functional GitHub repository][repo]. I must assume that you, as a developer, possess the following qualities:

1. Intermediate understanding of JavaScript.
2. Intermediate understanding of HTML.
3. Basic knowledge of client-server communication.
4. Basic knowledge of JSON.
5. Basic knowledge of node.js.

If you lack any of these qualities, you _may_ still be able to navigate this article and the related code, but these gaps in your knowledge will likely prevent you from extending my code to support a more realistic or non-trivial scenario. The internet is full of great resources that will provide you with the concepts necessary to master each of these items, and I encourage you to seek them out as needed - they are only a quick Google search away.

The current stack at [Widen][widen] has traditionally consisted of Java on the server, AngularJS for all of our browser-related code (within the last few years), Jersey for REST API support, and a whole host of other various libraries such as jQuery, underscore, lodash, jQuery UI, and Bootstrap. When designing the underlying sample web application, which I will be discussing shortly, I had four specific goals in mind:

1. **A _new_ and shiny approach**. Instead of developing yet another AngularJS-based UI, or deferring to jQuery, or creating a Java-based endpoint server using Jersey, or doing all three, I really wanted to make use of an entirely new set of tools. I hoped that this would allow me to gain a new perspective and evolve a bit more as a developer.

2. **Simplicity** is another desire of mine. I very much have grown to dislike the substantial learning curve associated with AngularJS 1.x, and am disappointed to discover that the learning curve for v2 is even more steep. The same is true of Java, which I have traditionally used server-side. I'd like to avoid as much boilerplate code as possible and get my application up and running fast without sacrificing scalability. Being able to easily describe my frontend as a collection of standalone focused components is also part of this goal. Also, traditional REST APIs are awkward to maintain and evolve. The frontend developers must coordinate with the backend developers to expose a set of API endpoints that properly support the browser-side representation of the model. As the needs of the UI change, this often requires the API to change as well. Surely there must be a better approach!

3. Some of the issues associated with a traditional REST API include unnecessary request overhead, large number of requests, and needlessly large response payloads. I was less concerned about client-side rendering performance, which React and AngularJS both handle fairly well, though Angular is much more monolithic and complex, making it easier to unknowingly introduce serious performance issues into an app. So, **efficiency** is my fourth goal.

4. Finally, I was looking for some approaches or tools that allow me to write uncharacteristically **elegant** code. The code itself should be easy to follow. Looking up and changing data from the UI should be intuitive. Ideally I would like to think about my model, and only my model - not in terms of available API endpoints. I'd also like to avoid much of the noisy boilerplate that is required of my traditional stack.

In order to address each of these goals, I decided to replace my current lineup of tools with an _entirely_ new set, some of which I  had never used before. This was very much a learning experience, one which I would like to share with you. In fact, some of Widen's emerging software products will make use of all of the new technologies discussed in this article. Next, I'll document each notable new tool. After the stack is clear, I'll walk you though, from start to finish, creation of a simple web application that is both functional and idiomatic of all items involved in this new stack.


## A futuristic stack

Adopting a completely new set of tools and architecture often means changing your perspective as a developer. Over time, we've all become comfortable with one or more tools. Whether that be jQuery or Angular, or Ember, or even the concept of REST, we have learned to trust and depend on our stack. We've been trained to think of web applications in a specific context though inculcation. Abandoning our stack and moving out of this comfort zone can be frustrating. Some of us may fight this urge, dismissing new choices as unnecessary or overly complex. Admittedly, I had the same thoughts about React, webpack, and Falcor before I had a strong understanding of them. In this section, I will briefly discuss each of the more notable tools in this futuristic stack.

### React

React differs from Angular and Ember due to its limited scope and footprint. While Angular & Ember are positioned as frameworks, React mostly concerns itself with the application "view". React contains no dependency injection or support for "services". There is no "jq-lite" (Angular) nor is there a required jQuery dependency (Ember). Instead of handlebars (Ember) you write your markup alongside your JavaScript using JSX, which compiles down into a series of JavaScript calls that build your document through React's element API as part of a "virtual DOM" that React maintains. It updates the "real" DOM from this virtual model in the most efficient way possible, avoiding unnecessary reflows/repaints, as well as delegating event handlers for you (among other things). If you embrace JSX (and from my experience, you should) you are adding a compilation phase to your project. For me, this was something I have tried to avoid for a while, but made my peace with this workflow after realizing how elegant and useful React through the lens of JSX really is. At this point, the floodgates opened up, and other useful JavaScript preprocessors, such as webpack and babel, were easy to embrace. More on those later.

In short, I really appreciate the relatively narrow focus of React. Dividing up a complex application into smaller components is something I grew to love with Angular. I was excited at the possibility of native support in the form of the Web Components spec, but ultimately chose React for its elegance, ease-of-use, small footprint, and relative maturity.


### Falcor

Falcor, a _very_ new library created and open-sourced by Netflix, is a complete departure from the traditional REST API. Instead of focusing on specific endpoints that return a rigid and predetermined set of data, there is only _one_ endpoint. This is how Falcor is commonly described, and it is a bit misleading, though technically correct. Instead of focusing on various server endpoints to retrieve and update your model data, you instead "ask" your API server for specific model data. Do you need the first 3 user names in your customer list along with their ages? Simply "ask" your API server, in a single request, for this specific data. What if you want only the first 2 user names and no ages? Again, a single request to the same endopint. The differences in these two GET requests can be seen by examining their query parameters, which contain specifics regarding the model properties of interest. Server-side, a handler for a particular combination or pattern of model properties is codified as part of a "route". When handling the API request, the falcor router (server-side) contacts the proper router function based on the items present in the query string.

It expects you to be able to model your data in as a [JSON graph][falcor-graph]. Your underlying data source does not necessarily have to maintain all data as a JSON graph, and in most cases it probably will not, but your Falcor API endpoint must translate your data into this format when responding to a request from your client. The necessity of this structure will become clearer as you go through this article and read a bit more about Falcor. The importance of organizing your data as a JSON graph will become even more obvious once you begin to explore [reference routes][falcor-ref], which I have left out of this post in order to keep it simple.

Falcor promotes a more intuitive API that _is_ your model. It also ensures that extra, unnecessary model data is never returned, saving bandwidth. Furthermore, requests from multiple disparate browser-side components are combined into a single request to limit HTTP overhead. Data is cached by Falcor client-side, and subsequent requests for cached data avoid a round-trip to the server. This decoupling of the model from the data source, along with all of the efficiency considerations, is exceptionally appealing. But the underlying concepts can be a bit mind-bending. I was a little confused by Falcor until I watched [this video by Jafar Husain][why-falcor-video], Falcor's lead developer.


### Webpack

Webpack, a build-time node.js library, further supports modularization of small focused React components. It also allows you to easily minify and concatenate your CSS and JavaScript, along with generation of source maps that make debugging much easier. After installing webpack and setting a few configuration options, it will monitor your code and generate new "bundles" whenever you make changes. Instead of importing a bunch of CSS or JS files client-side, you can simply import the bundle or bundles (depending on your configuration) generated by webpack, saving unnecessary HTTP requests on page load. Webpack also has a large number of plugins that can be used to "influence" the bundles it generates. For example, JSX is turned into JavaScript using the "jsx-loader" plugin. If you wish to write ECMAScript 6 code, but don't plan to limit support to browsers that fully implement the spec, you can use the "babel-loader" plug-in to ensure your ES6 code is turned into ES5-compliant code as part of webpack's bundle generation process.


### ES6

ECMAScript 6, also known as ECMAScript 2015, is currently the latest specification for the JavaScript language. It defines some big new features, such as fat arrow functions, classes, string interpolation, and the ability to create block scope. While ES6 support is limited to the newest browsers, it presents a host of elegant solutions and syntax that many developers find appealing and useful, myself included. Before ES6, developers that wanted to make use of some of these features had to settle for CoffeeScript or TypeScript. But now, all of the niceties in these higher-level abstractions are available in the underlying language itself. Horray! And if we want to ensure our ES6 code is executable in older browsers, a compiler named [babel][babel] can be used to transform your ES6 code into ES5 code as part of a build step. Once all of your supported browsers fully implement the specification, you can simply remove this build step.


## Building a simple app

To demonstrate all of the new items outlined above, I've created a trivial single-page application. While this is a contrived example, it exists to help you better understand how all of these technologies work together on a basic level. From this knowledge, you should be able to expand upon my code and build something a bit more realistic. Our sample app will allow us to read and modify a list of names. The list is maintained on the server, which will contain an initial list of names to be displayed to the user on page load. Changes to the list are initiated in the browser and persisted back to the server.

### Setup

Let's start by dividing up this application into logical pieces. At the most basic level, we have 2 segments - a client and a server. Our client exists entirely in the browser, and our server is a simple API endpoint. On the client, we must expose an interface that will allow our users to see and manipulate the names list. This list will be represented by a model that is understood by both the client and server. Essentially, our model will be expressed in JSON format in our "database", and will consist of an array of objects, with each object having a `name` property. While it is certainly _not_ a requirement for our model to be expressed in JSON format in our backing storage system, this will make our example app a bit easier to setup and understand.

Another important step, before diving into the code, is to think of our application in terms of reusable components that are not dependent on each other (wherever possible). Client-side, I can see three components - one that lists the names, another that allows names to be added, and a third that ties these two components together. This third component is important, as it allows us to forgo coding an explicit dependency between the list and "add name" components. Of these three components, at least two are reusable outside of our application.

The server itself can be divided up into multiple pieces. At the highest level, we have an HTTP server that exposes static resources and routes API requests. We will also need a series of code blocks that service the various model requests from our Falcor client. We can perhaps think of each such request handler, or route, as a separate piece. Finally, we have our backing store of data which also provides initial values. Our API endpoint handlers will delegate to this backing store for lookup and persistence of data.

I'll go over all of the dependencies in the following sections, but our most basic libraries and frameworks consist of:

1. React
2. Falcor
3. Express
4. Babel
5. Webpack
6. Node.js

You can see all dependencies for this sample application in the project's [package.json file][package.json].


### Creating a server

Please use the [server.js file][server.js] in the project's GitHub repository for reference as I discuss the server-side portion of our example app. Our server will handle API requests and serve up static resources (such as our JavaScript and HTML files). It will be written in JavaScript using Node.js, and will rely on Falcor and Express.


#### Getting started

We will represent our server using a single JavaScript file for simplicity. The first logical step is to reference all of our dependencies:

```javascript
var FalcorServer = require('falcor-express'),
    bodyParser = require('body-parser'),
    express = require('express'),
    Router = require('falcor-router'),
    app = express()
```

There are two Falcor-related dependencies. The first, `FalcorServer`, will be used to forward API requests to the most appropriate handler. The `Router` is what we will use to _define_ all these handlers. Each handler will be tied to a route string, which defines the type of model data associated with the request. While this may all seem a bit mysterious now, it will become clearer before the server-side section is complete.

The other three dependencies help us to serve static resources and parse HTTP requests into a more manageable form. For example, `express` listens to all HTTP requests on a specific port and either serves up static resources (such as our JS and HTML files) or routes the request to a more specific handler, such as our Falcor `Router`.

Let's also define our data store. Again, for simplicity, we will maintain our data directly in our node server via a simple JavaScript object:

```javascript
var data = {
  names: [
    {name: 'a'},
    {name: 'b'},
    {name: 'c'}
  ]
}
```

The above store also contains an initial list of names to be displayed to the user.


#### Request handlers

Let's continue by configuring our request handlers:

```javascript
app.use(bodyParser.urlencoded({extended: false}));
app.use('/model.json', FalcorServer.dataSourceRoute(() => new NamesRouter()))
app.use(express.static('.'))
app.listen(9090, err => {
    if (err) {
        console.error(err)
        return
    }
    console.log('navigate to http://localhost:9090')
});
```

In the first line, we're asking the `bodyParser` library to take any requests that contain application/x-www-form-urlencoded message bodies and parse the contents into a JavaScript object. This object is passed along as [a `body` property on the `Request` object][request.body] maintained by express. In our application, POST requests, which add new names to the list and contain URL-encoded name data, will be parsed by this handler before being forwarded on to a more specific handler by express.

The second line instructs express to delegate _any_ request for the "model.json" endpoint to our Falcor router, which we have not yet defined (we will soon). So, a GET request to "http://localhost:9090/model.json" will be handled here, as will POST requests to the same endpoint. For POST requests to this endpoint with a URL-encoded body, the body will first be parsed by `bodyParser` before being forwarded on to our router.

The third line serves up any static resources in the root of our project. While this wildcard is probably not appropriate for production, it is sufficient for this type of simple demo. Ideally, you should restrict access to static resources instead of allowing all source files to be served up.

Finally, the above code instructs express to listen to all HTTP requests on port 9090. This is the port that you must use client-side for access to static resources, as well as the API. Notice that we are making use of some ECMAScript 6 syntax here, specifically an [arrow function][es6-arrow]. In this case, an arrow function is simply a more elegant way of expressing a function argument. In ES5 syntax, our listener function is equivalent to:

```javascript
app.listen(9090, function(err) {
    if (err) {
        console.error(err)
        return
    }
    console.log('navigate to http://localhost:9090')
});
```

#### API route handlers

Next, we will create routes for three different API requests.


##### Number of names

First, we can expect our client to ask for the number of names in the list. The necessity of this information will become clear when we define our next route. The Falcor route that supplies the number of names in our list looks like this:

```javascript
var NamesRouter = Router.createClass([
  route: 'names.length',
  get () => {
    {path: ['names', 'length'], value: data.names.length}
  }
]);
```

Remember the `NamesRouter` we referenced in the previous section? This is our Falcor request router, and express will forward all appropriate API requests here. Our length route is simple, but exposes some Falcor-specific syntax that I have not yet explained. The `route` property identifies the "signature" of the request. If our Falcor client asks for the length of all names, the server-side Falcor router will match on this route string and execute the `get` function, which will describe the data to be returned to our Falcor client.

The response consists of two properties. The first such property, `path`, in our case, simply mirrors the route signature. The second property in our route's `get` handler is `value`. As you might expect, this holds the actual number of names in our list. We're simply pulling this value by checking the `length` property on the array that makes up our backing data store.


##### Display names for `n` name records

When our page loads, we will want to display at least some of the names in our database to the user. At this point, we don't have a concept of name records, name record IDs, or any other metadata associated with these names. Our goal is simple: display names to the user. So, how do we do this? First, we ask our server for the total number of names in our database. Then, we can construct a query that returns names, given a range. Say we want _all_ of the names in the DB, we can ask the server for name values starting with index 0 and ending with number of names - 1. I've already showed you what the "number of names" route looks like. Below is the route that returns the actual names, given a range parameter.

```javascript
{
    route: 'names[{integers:nameIndexes}]["name"]',
    get: (pathSet) => {
        var results = [];
        pathSet.nameIndexes.forEach(nameIndex => {
            if (data.names.length > nameIndex) {
                results.push({
                    path: ['names', nameIndex, 'name'],
                    value: data.names[nameIndex].name
                })
            }
        })
        return results
    }
}
```

Above, the `route` property identifies this route as one that will be invoked if the client requests a range of names. More specifically, a client request that is interested in only the `name` property of one or more name records. Our route handler generates an object containing a `path` and `value` property for each index in the range parameter.

For example, if we request the first two names in our DB, and these two records have respective `name` properties of "Joe" and "Jane", then our route handler will generate an array that looks like this:

```javascript
[
    {
        path: ['names', 0, 'name'],
        value: 'joe'
    },
    {
        path: ['names', 1, 'name'],
        value: 'jane'
    }
]
```

This will be returned to the Falcor response handler client-side, which will result in an update of the model cache. The caller will be provided with these names as well.


##### Add a new name record

Our simple names widget also allows us to add new names. I'll cover the client-side portion of this operation shortly. First, take a look at the server-side Falcor route:

```javascript
{
    route: 'names.add',
    call: (callPath, args) => {
        var newName = args[0];

        data.names.push({name: newName})

        return [
            {
                path: ['names', data.names.length-1, 'name'],
                value: newName
            },
            {
                path: ['names', 'length'],
                value: data.names.length
            }
        ]
    }
}
```

This is an example of a [Falcor "call"][falcor-call] route. The client will include "names.add" as the path parameter, along with the name to add, as part of a "POST" request. The endpoint described above will be hit, resulting in a new name in our DB. That is expected and straightforward, but the response to this request is interesting. Notice that we are returning two path elements to the client - both which describe changes that have occurred to our data set as a result of this new name. The first item in the set indicates that there is a new name added to the end of our names collection. The second item indicates that the number of names in our set has changed. If we were to add a new name of "Bob" to our existing names list of "Joe" and "Jane", the response generated by this route would look like this:

```javascript
[
    {
        path: ['names', 2, 'name'],
        value: 'Bob'
    },
    {
        path: ['names', 'length'],
        value: 3
    }
]
```

So the client doesn't have to ask the server about the index of this new name or the length of the names collection later on - it will be cached by Falcor client-side thanks to the information provided in the response to this "call" request. For small or simple data sets, it may make sense to return the changed paths _and_ values, but for larger data sets it may be advisable to _only_ return the changed paths. Your client may not care about _all_ of the changed values, and this may unnecessarily waste bandwidth and processor cycles. Instead, you can simply return the changed paths, and the Falcor client will know to ask the server, instead of the cache, when/if your application needs a value that is associated with one of these changed paths.


### The client

Our server code is quite simple - it serves up static resources, such as our JavaScript and HTML files, _and_ it responds to API requests from our client using Falcor. Next, I'll explain the client-side portion of our app, which, of course, runs in the browser.


#### Simplicity in our index page

[Our index page][index.html] is _very_ simple: just the usual HTML-related structuring along with one line to serve as the container for our entire React-generated app, followed by a second line that imports _all_ of our JavaScript.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
    <div id="demo"></div>
    <script src="/site/bundle.js"></script>
</body>
</html>
```

The only interesting lines reference the `<div>` and `<script>` elements. I can tell you that we _will_ compose the source of our app among multiple JavaScript files, but in the end, we will only serve up one file that contains all of our code. There is some overhead associated with every HTTP request, so reducing the number of requests on page load is beneficial.

Perhaps you are wondering why the `<script>` tag is listed at the bottom of the document, instead of inside of the `<head>` tag, as is customary. First, this allows the static markup to be loaded and displayed to the user immediately, instead of after all of our JavaScript source is loaded and parsed. In this case, there isn't much to speak of in terms of initial static content, but we could certainly add something that sets up the page, or perhaps even a "loading" message. Also, placing the script tag below the container element ensures this element is already available in the DOM by the time our code executes. Since our code will render all dynamic content inside of this container element, this is important.


#### Dividing UI roles into components with React

We can divide the frontend of our application into three logical components: a "name adder", a "names list", and a component that ties these two standalone components together. Each of these will be represented as self-contained React components. As you might imagine, each of these may need some way to communicate with our server. We'll make use of Falcor for that common task.


##### Using Falcor to communicate with our server

Falcor will not only make it easy for us to communicate with our server, it will also manage our data model and ensure that all trips to the server are both efficient and prudent. Our entire Falcor "helper" can be created with a few lines of code. We'll do this in a [model.js file][model.js]:

```javascript
var Falcor = require('falcor'),
    FalcorDataSource = require('falcor-http-datasource'),
    model = new Falcor.Model({
        source: new FalcorDataSource('/model.json')
    })

module.exports = model
```

If you are not very familiar with Node.js or its native module system - CommonJS - at least a few lines in the above code fragment may seem mysterious. The first two lines "import" Falcor and Falcor's HTTP data source modules. We will need these to setup our Falcor "helper". The last line in our file essentially creates a new module. This module represents our client-side Falcor model/helper, and can be `require`d by other modules that need to query the model through Falcor. Our exported module will be an object that has all of the properties defined in [Falcor's DataSource interface][falcor-ds]. The methods on this interface will be used by our React components to communicate with our model. You'll see how that works soon.

Our `model` is defined to be a wrapper around a Falcor HTTP `DataSource`. When defining the `DataSource`, we're including a path to our API server endpoint - "/model.json". For all calls to our API, we have _one_ HTTP endpoint. The type of operation and associated data is encoded as query parameters by Falcor for GET requests and the message body for POSTs.     


##### Names list component

Now that we have our model defined, let's start building up our UI with a component that lists all of the names in our store. This will be a React component. We'll keep it simple and implement it as a simple HTML list. This code is housed in a [names-list.jsx file][names-list.jsx].

```javascript
var React = require('react'),
    model = require('./model.js');

class NamesList extends React.Component {
    constructor() {
        super()
        this.state = {names: {}}
    }

    componentWillMount() {
        this.update()
    }

    render() {
        var names = Object.keys(this.state.names).map(idx => {
            return <li key={idx}>{this.state.names[idx].name}</li>
        })
        return (
            <ul>{names}</ul>
        )
    }

    update() {
        model.getValue(['names', 'length'])
           .then(length => model.get(['names', {from: 0, to: length-1}, 'name']))
           .then(response => this.setState({names: response.json.names}))
    }
}

module.exports = NamesList
```

In the first two lines, we're importing the React module, for obvious reasons, along with the Falcor model we defined in the previous step. If you are a Java developer, the component definition looks surprisingly familiar. ECMAScript 6 brings classes to JavaScript, and we're defining our names list component to be a type of React component. Again, similar to Java, we must define a constructor. We'll simply initialize our `state` object in this constructor. The `state` object will be used to feed data to our rendered markup, which will be re-rendered (as efficiently as possible by React) whenever it changes. Note that we _must_ invoke the `React.Component` constructor by calling `super()` _before_ we can access the context of our component. We access the context of our component using the `this` keyword.

Our first class method, `componentWillMount,` is inherited from the `React.Component` class. It will be called by React _just before_ when our markup is first "rendered" by React. That is, before the `render` method is invoked for the first time and the markup has been added to the DOM. At this point, we're calling the `update` method that grabs the list of names from Falcor.

The `update` method performs a few operations. First, it asks Faclor for the number of names in our list. Then, it sends a request for all names, given the result of the previous length request. Each of these calls returns a promise, since they are asynchronous. When the first call to get the number of names is resolved, our first `then` function is called with the result - the number of names in our list. Once we know that number of names, we go on to ask Falcor for all names between 0 and the last index of our list.

There is an implicit `return` keyword in our single-line ES6 arrow functions - each of these returns a [`Promise`][promise-mdn]. Once the second promise is resolved for our names list request, the next and final handler in our chain of model operations is invoked. The call to our length route was made using `getValue` which results in a single value as the resolved response (in this case, the number of names in our list). But our call to retrieve all names in the list is a `get`, which will return a JSON response containing all matching name objects with the specific property, `name`, filled in with a value. Notice we are calling `setState` with this collection of names. This updates our component's state object and instructs React to re-render the component with the new list of names.

Moving on to the `render` method - this is where the actual HTML elements are rendered to the DOM. React calls this when our component first mounts, and then again whenever our component's `state` property changes. The `names` property, part of our component's state, is an object with keys that represent the index of each name on our server and values containing each name record. Since we only asked Falcor for the `name` property in each record, that is the only property we will find in returned name record.

The markup in our `render` method may look a bit strange - it's JSX, which is an extension to the ECMAScript language specification created and maintained by Facebook. It allows you to easily include HTML-like content alongside JavaScript code. Before it is delivered to the browser, we will have webpack compile this JSX down to standardized JavaScript. More on that later. The result of this build step will look like a bunch of method calls that build up out HTML. We could have taken that approach as well and built up our HTML using React's DOM API instead of using JSX, but JSX makes our lives a _lot_ easier and the code much simpler to follow.

The last line in our file allows our NamesList component to be pulled into another module and actually used. We'll do just that very soon.


##### Name adder component

We have a component that will list our names, but we also want to be able to add new names to our list. So, next we will need to create a "name adder" React component, which will be stored in a file appropriately called [name-adder.jsx][name-adder.jsx].

```javascript
var React = require('react'),
    model = require('./model.js');

class NameAdder extends React.Component {
    handleSubmit(event) {
        event.preventDefault()

        var input = this.refs.input

        model.
            call(['names', 'add'],
                [input.value],
                ["name"]).
            then(() => {
                input.value = null
                input.focus()
                this.props.onAdded()
            })
    }

    render() {
        return (
            <form onSubmit={this.handleSubmit.bind(this)}>
                <input ref="input"/>
                <button>add name</button>

            </form>

        )
    }
}

NameAdder.propTypes = {
    onAdded: React.PropTypes.func.isRequired
}

module.exports = NameAdder
```

In our `render` method, we're defining a simple HTML form that contains a text input and a submit button. The user is able to submit the form either by clicking the submit button or hitting the enter key after typing text into the input field. When the form is submitted, the `handleSubmit` method in our component class is called, passing the submit [`Event`][event-mdn]. Notice we are creating a new function that binds the component as context. This is needed when utilizing ES6 classes in React components. Otherwise, the value of `this` inside our event handler will be `window` in this case, which is _not_ what we want.

The other method in our React component is `handleSubmit`, which is called when our rendered form is submitted, as I mentioned above. First, we must prevent the browser's default action. In other words, we don't want to _actually_ submit the form; we don't want the page to reload. Instead, we need to funnel the submitted data to Falcor. Next, we must look up our input element. We'll need this to determine what text the user entered. Notice that we included a `ref` attribute on the text input in our `render` method. This allows us to easily get a handle on the underlying DOM element without resorting to a CSS selector. Finally, we must send this new name to our server. We want to hit the "names.add" call route we defined earlier, passing the new name. Once our server persists the new name and responds, Falcor will update its internal representation of our model using the information provided by our server. It now knows that there is one more name in our list, and it knows the index of the name we just added. But why is this important?

After Falcor has determined that the name has been successfully added to our server, it will invoke our "success" function, which is the first (and only) function we have passed when calling `then` after invoking `call` on our Falcor model. This gives us the opportunity to reset our text input and ensure it retains focus so that our user can easily enter a new name. But we also want to be sure our list of names is current. It looks like there is an `onAdded` function on a `props` property attached to our component. Where did that come from? The component that rendered our name adder component passed this to us, which we will see next. Any parameters passed to a React component are available on the `props` property. We can expect that an `onAdded` function is passed to our component, and we should always invoke it when a new name has been added. I can tell you now that this function will trigger the `update` method on our `NamesList` component, which, as you might remember, will result in a call to Falcor for our list of names. This is exactly what we want to do - update our list of names after a name is added so our user sees the current list. You might be surprised to know that, after adding this name, Falcor does _not_ contact our server for this list of names. It already knows exactly how the list has changed, thanks to the information provided by our server's response to the "names.add" call. It pulls this data from its internal representation of our model, saving a couple round-trips to the server (one for the length request, and another for the list of names).

Finally, we are making use of [React's property validation][react-prop-validation] feature. Have a look at the line at the end of the file that starts with `NameAdder.propTypes = {`. If the component that renders our `NameAdder` does not pass a callback function property to our component, React will log a warning message to the developer console in your browser. This is a useful way to alert any developers integrating your component when they have inadvertently omitted a vital property. Defining these property validations in your component also serves as a form of documentation.

Perhaps you are starting to see the elegance of this modern stack. React allows us to compose our UI in terms of focused components, and Falcor lets us think about our model in terms of the actual model properties, all while ensuring that communication with the server is minimized.


##### Name manager component

We have a component to list all of our names, and another to add a new name. These two components don't have any direct knowledge of each other. This is a good thing, as it makes them easier to test and re-use. But we still need some way to tie these two components together. The solution: a "glue" component. We'll call this third React component `NameManager`, stored in a [name-manager.jsx][name-manager.jsx] file.

```javascript
var React = require('react'),
    ReactDom = require('react-dom'),
    NameAdder = require('./name-adder.jsx'),
    NamesList = require('./names-list.jsx');

class NameManager extends React.Component {
    handleNameAdded() {
        this.refs.namesList.update()
    }

    render() {
        return (
            <div>
                <NameAdder onAdded={this.handleNameAdded.bind(this)}/>

                <NamesList ref="namesList"/>
            </div>

        )
    }
}

ReactDom.render(<NameManager/>, document.querySelector('#demo'))
```

As expected, we must first import `React`, our `NameAdder`, and `NamesList` components. We'll also need ReactDom, which we must use to render our finished UI into the DOM. Note that we are selecting the container element we defined in our [index.html][index.html] file earlier, and rendering our entire set of React components as children/descendants.

The `render` method, which is called when `ReactDom` attempts to render our component to the DOM, is largely a set of references to the two other components we already defined. Remember how our `NameAdder` component was able to ask the `NamesList` component to update its list of names? This is made possible by our `NameManager` component. You can see that is passed a property, `onAdded`, to this component. When it is called by `NameAdder`, the `handleNameAdded` method is called on `NameManager`, which in turn delegates to the `NamesList` component's `update` method, which has been exposed as a public class instance method.

And that's about it for our React components. Pretty simple, eh? The next section will cover how webpack allows us to build our frontend components into a single bundle file, which will be usable in all of our supported browsers.


#### Modularizing our components and simplifying the build process with webpack

We'll use webpack as a build tool to accomplish a few goals:

1. Compile our JSX to standardized JavaScript.
2. Combine all needed JavaScript into a single file.
3. Ensure our ES6 syntax works in all browsers, regardless of the completeness of their implementation of the specification.
4. Ensure debugging our code in the browser is simple by providing access to the original pre-compiled/combined source files.

We have defined webpack, along with all other dependencies, in a [package.json file][package.json]. All that is left is a bit of configuration. Have a look:

```javascript
module.exports = {
    entry: './name-manager.jsx',
    output: {
        filename: './site/bundle.js'
    },
    module: {
        loaders: [
            {
                test: /\.js/,
                loader: 'babel',
                exclude: /node_modules/
            }
        ]
    },
    devtool: 'source-map'
}
```

If we name the file [webpack.config.js][webpack.config.js], webapck will be able to easily discover and use our configuration. The main entry point of our app, "name-manager.jsx" is used as the value of the `entry` configuration property. Webpack will use this "main" class to find all other project dependencies, which it will use to generate the final combined JavaScript file imported by our index.html page. The name of that combined file is set on the `output.filename` config property.

Next, a set of "loaders" are specified. We're using the babel loader, which ensures ECMAScript 6 code is compiled down to ECMAScript 5 syntax, which allows us to write purely ES6 code without having to worry about which portions of the spec our target browsers support. The `test` property on our loader is a regular expression, and it results in webpack passing any .js or .jsx files in our source tree to the babel loader for processing. The babel loader processes our source before webpack combines everything to a single resource.

Finally, the last line of our configuration instructs webpack to generate source maps. This satisfies #4 in our list of goals. Source maps are only loaded by the browser when the developer tools console is open, so you don't have to worry about wasting bandwidth on page load when your users visit the app. These maps allow us to see the original source files, and even set breakpoints anywhere in these files. We don't even have to look at the combined and compiled bundle.js file. Webpack will annotate the bottom of the generated bundle.js file with a pointer to the source map file, so our browser's dev tools know how to find it. This becomes even more useful when we generate a minified bundle file for use in production. While I'm not generating a minified bundle in the example webpack config, you can easily do this simply by running webpack with a `-p` command-line option. The "p" is short for "production".


### Building and using our app

All of our code is in place, our server is ready, _and_ we have a build tool in place. How do we get our app up and running?

First, we need to pull in all project dependencies. In the root of our project, we simply run `npm install`, which will parse our [package.json file][package.json] and install all registered dependencies inside of a "node_modules" folder. This will also install a webpack binary, which we will need to build the client-side source bundles.

The next step is to generate the source bundle referenced by our index.html file. All we need to do here is to run webpack by executing `$(npm bin)/webpack` from the root of our project. `$(npm bin)` expands to the directory that contains all binaries pulled in by `npm install`. Note that this path expansion only works in linux/unix environments. A cross-platform option involves adding a reference to the webpack binary inside of your [package.json file][package.json]. For example, if you include the following `"scripts"` property, you can run webpack _and_ allow it to re-generate your bundles whenever code changes on _any_ platform by running `npm run webpack`:

```javascript
"scripts": {
    "webpack": "webpack -w"
}
```

Finally, we need to start our web server, which handles API requests and serves up our index.html, bundle.js, and source map files. To start the Node.js server, execute `node --harmony server`. The `--harmony` switch tells node that we are using syntax included in the newest ECMAScript specifications, known as "harmony". ECMAScript 6 is one such entry in the harmony set of specifications. You may not need this switch if you are using a very recent version of node.js.

After starting up the server, our app will be accessible on port 9090. So, navigate to http://localhost:9090 and test it out!


## Going further

There's a lot more we can do with Falcor, React, Webpack, and ECMAScript 6, of course. This post and the associated example exists simply to get you started. In particular, you should read up on a Falcor topic that I omitted from my example for the sake of simplicity: [reference routes][falcor-ref]. When you develop a _real_ web application backed by non-trivial data, you'll find yourself making use of the reference type in Falcor quite often. I encourage you to spend some time following the Falcor and React tutorials as well. And if you need more information regarding ECMAScript 6, [Mozilla Developer Network is a _great_ reference][es6-mdn].

Here are some ways that you can improve our simple names app, if you are interested in further honing your skills:

1. Allow existing names to be edited. This will require an "edit names" React component, as well as a Falcor "set" route.
2. Allow existing names to be re-ordered. You'll probably need to add code to the NamesList component, along with another Falcor route to handle index updates.
3. Support name deleting. This is best handled by an additional React component and a new "call" route in the Falcor backend.

Feel free to issue pull requests to the underlying [GitHub repository][repo] if you'd like to share your changes and additions.


[babel]: https://github.com/babel/babel
[es6-arrow]: http://www.ecma-international.org/ecma-262/6.0/#sec-arrow-function-definitions
[es6-mdn]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/New_in_JavaScript/ECMAScript_6_support_in_Mozilla
[event-mdn]: https://developer.mozilla.org/en-US/docs/Web/API/Event
[falcor-call]: http://netflix.github.io/falcor/doc/DataSource.html#call
[falcor-ds]: http://netflix.github.io/falcor/doc/DataSource.html
[falcor-graph]: http://netflix.github.io/falcor/documentation/jsongraph.html
[falcor-ref]: http://netflix.github.io/falcor/documentation/jsongraph.html#reference
[index.html]: https://github.com/Widen/fullstack-react/blob/1.2.1/index.html
[model.js]: https://github.com/Widen/fullstack-react/blob/1.2.1/model.js
[name-adder.jsx]: https://github.com/Widen/fullstack-react/blob/1.2.1/name-adder.jsx
[name-manager.jsx]: https://github.com/Widen/fullstack-react/blob/1.2.1/name-manager.jsx
[names-list.jsx]: https://github.com/Widen/fullstack-react/blob/1.2.1/names-list.jsx
[package.json]: https://github.com/Widen/fullstack-react/blob/1.2.1/package.json
[promise-mdn]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
[react-prop-validation]: https://facebook.github.io/react/docs/reusable-components.html#prop-validation
[repo]: https://github.com/Widen/fullstack-react
[request.body]: http://expressjs.com/api.html#req.body
[server.js]: https://github.com/Widen/fullstack-react/blob/1.2.1/server.js
[webpack.config.js]: https://github.com/Widen/fullstack-react/blob/1.2.1/webpack.config.js
[widen]: http://widen.com
[why-falcor-video]: https://netflix.github.io/falcor/starter/why-falcor.html
