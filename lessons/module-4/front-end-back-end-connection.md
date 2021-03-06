---
title: Connecting Front-End and Back-End Applications
length: 90 mins
tags: javascript, backend
---

<!-- In introduction, say that learning this architecture will set themselves apart from other bootcamp graduates -->
<!-- Create backend to start out with instead of making it from scratch - have them walk through the app (connect the walkthrough to an interview question) -->
<!-- They were engaged with the CORS section - can they be divided in groups to find out what protocol, host, port is? Have them write it on the board while they are working - can utilize the back whiteboards as well (write down the questions on the boards)-->
<!-- Have them close computers for the diagramming portion - make sure they are taking notes! -->
<!-- Once they have drawn the diagrams and I have corrected what they have on the board, have them reassess their diagrams and force them to correct what they did - drives a deeper level of understanding -->
<!-- For the final summary piece, use ONLY notes that you took, summarize all the steps we took today -->

## Objectives

Create a back-end and front-end application in two separate repositories and allow them to talk to each other - even on production.

## Introduction

A common application architecture is to host a front-end (user-interfacing) application and back-end (data-serving) application separately from each other. The goal is separation of concerns and therefore an increased ease in continuous integration between separate teams. There are some local development and production environment issues that must be addressed for this to happen smoothly.

You might have done this before - an Express server and create-react-app application in one repository. We won't be doing that because it's against the architecture we're looking for.

We're also not going to deal with proxies today, and we certainly won't be using any Chrome extensions for CORS (if it's on now, turn it off).

## Create a Back-End

Create an Express application that has one endpoint (a root-level endpoint, `/`, is adequate) and serves a JSON object of arbitrary data (perhaps an array of objects). Don't forget your `.gitignore`.

Test the endpoint using Postman.

## Create a Front-End

If you have not done so recently, update your create-react-app npm package using the command: `npm i -g create-react-app`. Create a new create-react-app React front-end application - name it whatever you'd like.

Once your app is done being created, start your back-end application and your front-end application. What happened?

<!-- They should see that the FE and BE app are trying to run on the same port - change the BE development to be something like 3010 -->

*Setting a new back end port*

You probably saw an error having to do with both apps trying to run servers via the same port. Oh no! How do we fix it? The simplest solution is to edit the port for our BE server from `3000` to `3001`.

```js
app.set('port', process.env.NODE_ENV || 3001)
```

We edit this because `create-react-app` already defines a port for its server, and it is easier for us to just move our Express server to another port.

Now that that's fixed, create a `fetch` call in the application to fetch data from your back-end application:

* State should hold the data from the fetch call
* There should be some default state of the fetched data (the default state should be rendered on the page)
* Once the fetch is complete, the state should be updated and rendered on the page

If it's not working, look for an error in the console of your React app...

<!-- They should see a CORS error -->

You'll probably see something like this: `"Cross-Origin Request Blocked: The Same Origin Policy disallows reading the remote resource at $somesite"` - it's the dreaded CORS error!

### CORS! - Some Group Work

You've heard of it before - Cross-Origin Resource Sharing. What even is an origin? And why do we want to share resources?

With a partner, look through [this MDN page](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) about what an origin is. See if you can describe an origin for our use cases in 1 or 2 sentences.

---

Now that we know what an origin is, what is Cross-Origin Resource Sharing, and why won't it let our front-end talk to our back-end application?

Read through [this MDN page](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) about CORS - specifically the "Introduction", "What requests use CORS?", "Functional Overview", the **Simple requests** portion of "Examples of access control scenarios", and "The HTTP response headers".

Now in each group at your tables on a piece of chart paper:

* Diagram the "conversation" between the back-end and front-end that caused the CORS issue in the first place
* Then diagram a new conversation where the front-end is able to request a resource from the back-end, which is another origin (what needs to change?)

---

Let's fix the CORS issue.

*Fixing CORS errors in Express*

<!-- They need to add the "cors" express package and use the default app.use(cors()); in their server file -->

Let's install an extension designed to address Express CORS handling into our BE express app:

```
$ npm install cors --save
```

Then, in our `server.js` file, we can import it and create app-level middleware to ensure that our endpoints stop throwing CORS errors (by allowing all origins to access the resources).

```js
const express = require('express');
const app = express();

// add CORS
const cors = require('cors');

// add app-level middleware
app.use(cors());
```

Now that the CORS situation is fixed, and even though it works, note that you _SHOULD NOT use the default CORS settings_ to allow all origins! In your back-end application, you need to change the default settings in the server to allow only specific origins that apply to your application (for development and production).

You can read up on how to configure the CORS middleware in the [documentation here](https://expressjs.com/en/resources/middleware/cors.html#configuring-cors).

### Do Not Hard-Code Host Names...

In the fetch call of our FE application, we hardcoded the URL we are querying. But `localhost` won't work when we get to our production application. How can we make the URL dynamic?

Go to `create-react-app` docs for the section on [environment variables](https://facebook.github.io/create-react-app/docs/adding-custom-environment-variables#docsNav).

Take a few minutes to read through that section. Can you find out how to add a custom environment variable and any caveats about them?

<!-- Need to add them in some kind of .env file, in our case .env.development -->
<!-- Need to have prefix REACT_APP_ -->
<!-- NODE_ENV environment variable is available by default -->
<!-- The environment variables are embedded during the build time, not run time -->

*Making a custom create-react-app environment variable*

Let's add an environment variable for the `BACKEND_URL` in a `.env.development` file. Replace the hard-coded URL in your App fetch call with the environment variable, and see if it works!

In the root of your react app, create a dot-prefixed env file to store variables that will be used when we are in the `development` environment.

```
$ touch .env.development
```

Then, let's add a custom React environment variable to store our backend URL for the fetch call.

Look through the docs (linked above) to see how to format the environment variable.

```
REACT_APP_BACKEND_URL=http://localhost:3001
```

Now, in our FE application, let's update the hardcoded URL in our fetch to use the new environment variable.

```js
fetch(process.env.REACT_APP_BACKEND_URL + '/api/v1/[YOUR ENDPOINT HERE]')
```

You will have to kill your FE server and restart it. This is because, as you read in the documentation, environment variables are embedded into the code during the BUILD of the app, not during the run time. So now that we've added a new variable for our development environment, we must start a new build.

In the development environment, a new build is run every time we run `$ npm start`, so let's do that now. Make sure your BE server is already running.

Check your react app in the browser. Hopefully we will still be successfully making our fetch call using the environment variable!

### Deploy

So we're set up locally, and everything seems to be working correctly! So far we've taken care of our development environment. What about production and deploying our application?

Let's deploy the back-end application to Heroku first. Go ahead! (Reference the lesson we've already had about deploying a back-end app to Heroku.)

Test it with Postman to make sure the API is working.

---

Next we need to deploy the front-end. This is a little different from deploying the back-end. To deploy something to Heroku, we need a server. Our React app has a development server (this is what is used when you say `npm start` in your terminal), but we can't use it in production.

We have to use what is called a buildpack. It essentially wraps our React app in a server (like an Express server) for us, which Heroku can use to serve our front-end application.

[This is a buildpack for create-react-app](https://github.com/mars/create-react-app-buildpack) that is suggested. To set it up and create a Heroku application, run the command: `heroku create your-app-name --buildpack mars/create-react-app`

Push up your FE app and watch it build. Does it run? Does the fetch call go through successfully? What happened?

<!-- The fetch call will not go through because the BACKEND_URL has not been set for production through Heroku -->

You will probably see your default state information being rendered, but nothing from your fetch call. Why?

We only defined a backend url for our *development* environment, but on Heroku, we're in a *production* environment!

Read [this Heroku doc page](https://devcenter.heroku.com/articles/config-vars#using-the-heroku-dashboard) for what you might need to add.

In the settings, under 'CONFIG VARIABLES', add your `REACT_APP_BACKEND_URL` variable with a value of your deployed BE's URL.

Did it fix the issue?

Probably not! According to the docs, when are environment variables embedded? What will you have to do to get the newly created production environment variable into your codebase?

## Summarize What We Have Done

Take a few minutes now to write down, in your own words, exactly the steps you had to take to get your BE and FE to talk to one another. What caused bugs? How did you address those bugs?

Write down what remaining questions you have. Use those question as starting places for digging into the documentation.
<!--  -->
