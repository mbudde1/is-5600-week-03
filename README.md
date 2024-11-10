# Lab 03 | Node API

## Table of Contents

1. [Lab 03 | Node API](#lab-03--node-api)
   - [Overview](#overview)
   - [Instructions](#instructions)
2. [Guidance and Testing](#guidance-and-testing)
3. [Submission](#submission)
4. [Getting Started with GitHub and Codespaces](#getting-started-with-github-and-codespaces)
   - [Step 1: Fork the Repository](#step-1-fork-the-repository)
   - [Step 2: Open the Repository in Codespaces](#step-2-open-the-repository-in-codespaces)
   - [Step 3: Complete the Lab Assignment](#step-3-complete-the-lab-assignment)
   - [Step 4: Submit Your Work via Pull Request](#step-4-submit-your-work-via-pull-request)


## Overview

In this lab we will begin to build out a basic NodeJS API. We will be using the NodeJS `http` module to create a web server. We will then configure the server to return JSON data. We will also configure the server to use routing to determine how to respond to different requests.

Then we will replace the `http` module with Express, and create a very simple chat application.

Most of this lab is taken from your Fullstack NodeJS textbook which you were assigned in Week 2. 

The goal of the lab is to practice using NodeJS to create a simple API. 

## Instructions

1. Open up and examine the `app.js` file. This is the only file we will be working with for this lab. You will be adding code to this file to complete the lab.
2. First lest's begin by creating a basic web server. The server will simply return a single response.

```js
const http = require('http');
const url = require('url');

const port = process.env.PORT || 3000;

// note that typically the variables here are `req` and `res` but we are using `request` and `response` for clarity
const server = http.createServer(function(request, response) {
  response.end("hi");
});

server.listen(port, function() {
  console.log(`Server is listening on port ${port}`);
});
```

If you open the web view and navigate to the URL, you should see the "hi" message. In Codespaces, the browser should automatically open up and you should see your message.

3. Next, let's configure the server to serve some JSON. JSON is a common format for data exchange. It is a string that is formatted like a JavaScript object. We can use the `JSON.stringify` method to convert a JavaScript object to a JSON string. We can then use the `response.end` method to send the JSON string to the client.

```js
// modify the server code to look like this
const server = http.createServer(function(request, response) {
  response.setHeader('Content-Type', 'application/json');
  response.end(JSON.stringify({ text: 'hi', number: [1, 2, 3] }));
});
```

Refresh your lab browser, you should see the JSON string returned in the browser.

4. Now that we have a working server, and we have it configured to return JSON. We should introduce the routing. Routing is the process of determining how an application responds to a client request to a particular endpoint. Each endpoint is a unique URL. We are going to configure two endpoints or routes, one that will return our plain text message, and one that will return our JSON data. We will use the `request.url` property to determine which route to take.
First, let's define the functions that will be used to return the two responses. In the `app.js` file, define the following functions:

```js
/**
 * Responds with plain text
 * 
 * @param {http.IncomingMessage} req
 * @param {http.ServerResponse} res
 */
function respondText(req, res) {
  res.setHeader('Content-Type', 'text/plain');
  res.end('hi');
}

/**
 * Responds with JSON
 * 
 * @param {http.IncomingMessage} req
 * @param {http.ServerResponse} res
 */
function respondJson(req, res) {
  res.setHeader('Content-Type', 'application/json');
  res.end(JSON.stringify({ text: 'hi', numbers: [1, 2, 3] }));
}

```

We will also add a third function, this will be used to return a 404 response not found.

```js
/**
 * Responds with a 404 not found
 * 
 * @param {http.IncomingMessage} req
 * @param {http.ServerResponse} res
 */
function respondNotFound(req, res) {
  res.writeHead(404, { 'Content-Type': 'text/plain' });
  res.end('Not Found');
}
```

Now that we have these functions defined, we need to create a request listener that will call each one ending on the path of the request. We will use the `request.url` property to determine the path.

```js
const server = http.createServer(function(request, response) {
  const parsedUrl = url.parse(request.url, true);
  const pathname = parsedUrl.pathname;

    console.log("url", pathname);
    if (pathname === '/') return respondText(request, response);
    if (pathname === '/json') return respondJson(request, response);

  respondNotFound(request, response);
});
```

In your web viewer, visit the `/` and `/json` endpoints. You should see the appropriate response returned. If you visit any other endpoint, you should see the 404 response.

5. These responses are fine, but often a web server is responding to content that has been given to it. So we should create a dynamic response. We will create a route that will take an input, and then return the input in various formats via JSON object. The request will look like this: `/echo?input=fullstack` The object will return `fullstack` with various transformations about it. The object will have the following properties:

- `normal`: The input string without a transformation
- `shouty`: The input string in all caps
- `charCount`: The number of characters in the input string
- `backwards`: The input string reversed

First, let's modify our server function so that we listen for a matching query parameter:

```js
const server = http.createServer(function(request, response) {
    const parsedUrl = url.parse(request.url, true);
    const pathname = parsedUrl.pathname;  

    if (pathname === '/') return respondText(request, response);
    if (pathname === '/json') return respondJson(request, response);
    if (pathname.match(/^\/echo/)) return respondEcho(request, response);
    
    respondNotFound(request, response);
});
```

Next let's create `respondEcho` as an event handler:

```js
/**
 * Responds with the input string in various formats
 * 
 * @param {http.IncomingMessage} req
 * @param {http.ServerResponse} res
 */
function respondEcho(req, res) {
    const urlObj = new URL(req.url, `http://${req.headers.host}`);
    const input = urlObj.searchParams.get('input') || '';

    res.setHeader('Content-Type', 'application/json');
    res.end(JSON.stringify({
        normal: input,
        shouty: input.toUpperCase(),
        charCount: input.length,
        backwards: input.split('').reverse().join(''),
    }));
}
```

In your web viewer, visit the `/echo?input=fullstack` endpoint. You should see the appropriate response returned.

6. Now, let's introduce Express. Express is a fast un-opinionated web framework for Node.js. It is a very popular framework for building web applications. It is very easy to get started with, and it is very flexible. We will use it to build our web application. First, let's install it. In the Shell, run the following command:

```bash
npm install express
```

Once this has finished installing, we'll need to turn the server back on:

```bash
npm start
```

*note To learn more about NPM and why we use it [watch this video](https://www.youtube.com/watch?app=desktop&v=P3aKRdUyr0s)*


Now - let's modify our `app.js` file to use Express. First, let's import the module. We will keep the listener functions that we've already defined, but simply modify the rest of the file to look like this:

```js
const express = require('express');

const port = process.env.PORT || 3000;

const app = express();

// function declarations for respondText, respondJson, respondNotFound and respondEcho stay here

app.get('/', respondText);
app.get('/json', respondJson);
app.get('/echo', respondEcho);

app.listen(port, () => {
  console.log(`Listening on port ${port}`);
});
```



This is a great demonstration of separation of concerns and abstraction. By abstracting our listener functions out of the body of the `server` definition from our previous iteration, we are able to drop in a new server framework without having to change the logic of our application. We can simply drop in a new server framework, and then configure the routes. This is a great example of how to write modular code.

In your web viewer, visit the `/`, `/json`, and `/echo` endpoints. You should see the appropriate response returned.

7. Since express is a drop in replacement for the `http` module, we don't technically need to modify any of our listeners. However, we can make use of express's built in features to make our code more concise. Our `respondText` function is fine, and doesn't need to be changed. However, we can clean up our `respondJson`.

```js
function respondJson(req, res) {
  // express has a built in json method that will set the content type header
  res.json({
    text: 'hi',
    numbers: [1, 2, 3],
  });
}
```

Next, let's look at our `respondEcho` method:
  
```js
function respondEcho (req, res) {
  // req.query is an object that contains the query parameters
  const { input = '' } = req.query;

  // here we make use of res.json to send a json response with less code
  res.json({
    normal: input,
    shouty: input.toUpperCase(),
    charCount: input.length,
    backwards: input.split('').reverse().join(''),
  });
}
```

9. For the final section, we are going to add two simple endpoints to create a faux chat app. We already have an `chat.html` file in the repo. This file contains some simple HTML markup for the app. There is also a `public/chat.js` file - this file is empty, so we'll add the client side javascript there first.

```js
// public/chat.js
new window.EventSource("/sse").onmessage = function(event) {
  window.messages.innerHTML += `<p>${event.data}</p>`;
};

window.form.addEventListener('submit', function(event) {
  event.preventDefault();

  window.fetch(`/chat?message=${window.input.value}`);
  window.input.value = '';
})
```

This code is pretty simple. We are using the `EventSource` API to listen for messages from the server. When we receive a message, we append it to the `messages` element. We are also using the `fetch` API to send a message to the server. We are using the `GET` method to send the message as a query parameter.

Now we still need to build out our newly defined endpoints. Let's start with the `/chat` endpoint. This endpoint will be similar to our `/echo` endpoint. It will receive a message as a query parameter. Next the `/sse` endpoint will broadcast the message to all connected clients.

Lastly, because our app is running in Codespaces, we need to configure and endpoint to serve up our HTML file, `chat.html`. 

10. Let's start by wiring up the `chat.html` file. Add the following function to your `app.js` file:

```js
// app.js

/**
 * Serves up the chat.html file
 * @param {http.IncomingMessage} req
 * @param {http.ServerResponse} res
 */
function chatApp(req, res) {
  res.sendFile(path.join(__dirname, '/chat.html'));
}

// register the endpoint with the app (make sure to remove the old binding to the `/` route)
app.get('/', chatApp);
```
This will tell express to serve up the `chat.html` file when the `/` endpoint is hit. Next, let's add the `path` module to the top of the file:

```js
const path = require('path');
```

Lastly, we need to tell express that we have a `public` folder. The `public` folder is where apps typically serve what are called static assets, these can be images, css, or javascript files that are loaded by the client. We will use this folder to serve up our `chat.js` file. Add the following line to your `app.js` file:

```js
const app = express();
// add this line just after we declare the express app
app.use(express.static(__dirname + '/public'));
```

Great, now you should be able to restart you replit lab, and visit the `/` endpoint. You should see the chat app. If you open up the console, you should see the `chat.js` file being loaded.

11. Next, let's register the `/chat` endpoint.

```js
app.get('/chat', respondChat);

function respondChat (req, res) {
  const { message } = req.query;

  chatEmitter.emit('message', message);
  res.end();
}
```

If you notice in the code above, we are referencing a variable called `chatEmitter`. This is a new variable that we need to define. Let's add a require statement at the top of the file to import the `events` module.

```js
const EventEmitter = require('events');

const chatEmitter = new EventEmitter();
```

Now our app should be receiving messages via the `/chat` endpoint. We are not broadcasting the messages quite yet, but you can verify that the messages are being received by placing a `console.log` statement in the `respondChat` function.

12. We still need to tell the client that our server has received new messages. Let's register the `/sse` endpoint.

```js
/**
 * This endpoint will respond to the client with a stream of server sent events
 * @param {http.IncomingMessage} req
 * @param {http.ServerResponse} res
 */
app.get('/sse', respondSSE);

function respondSSE (req, res) {
  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Connection': 'keep-alive',
  });

  const onMessage = message => res.write(`data: ${message}\n\n`); // use res.write to keep the connection open, so the client is listening for new messages
  chatEmitter.on('message', onMessage);

  res.on('close', () => {
    chatEmitter.off('message', onMessage);
  });
}
```


And that is it! You should be able to press start in the lab, open one or more tabs to the `/chat.html` page, and send messages between the tabs. You can also open the `/chat.html` page in a new browser window and send messages between the tabs and the window.


## Guidance and Testing

1. The steps above should walk you through creating the layout to match the mockup. You can review the video walkthrough for further guidance.
2. If you're having trouble with the Simple Browser, click the refresh button 
3. At the end of each Lab Readme are a series of tips and reminder on how to create a pull request and start Codespaces.

## Submission

Once you have completed the lab, please submit your lab by committing the code and creating a pull request against the `main` branch of your forked repository.

Once you have a URL for your Pull Request, submit that URL with a brief message in Canvas against the Assignment. 

## Getting Started with GitHub and Codespaces

Welcome to the course! In this guide, you’ll learn how to set up your coding environment using GitHub and Codespaces. By following these steps, you’ll be able to work on your lab assignments, write and test your code, and submit your work for review. Let's get started!

### Step 1: Fork the Repository

Forking a repository means making a copy of it under your GitHub account. This allows you to make changes without affecting the original project.

1. **Open the Repository**: Start by navigating to the GitHub repository link provided by your instructor.
2. **Click "Fork"**: In the top-right corner, find the “Fork” button and click it.
3. **Select Your Account**: Choose your GitHub account as the destination for the fork. Once done, you’ll be redirected to your forked copy of the repository.

   > **Tip**: Make sure you’re logged into your GitHub account, or you won’t see the option to fork!

### Step 2: Open the Repository in Codespaces

With your forked repository ready, you can now set up a development environment using Codespaces. This setup provides a pre-configured environment for you to code in, with everything you need to complete the lab.

1. **Open the Codespaces Menu**:
   - In your forked repository, click the green "Code" button, then switch to the "Codespaces" tab.
2. **Create a Codespace**:
   - Click on "Create codespace on main" to start the setup.
3. **Wait for Codespaces to Load**:
   - It may take a few minutes for Codespaces to create and configure your environment. Be patient, as it’s setting up all the tools you’ll need.
4. **Start Coding**:
   - Once the setup is complete, Codespaces will automatically open a new browser tab where your code will be ready to run. You’ll be able to see the code and any outputs as you go through the lab assignment.

### Step 3: Complete the Lab Assignment

Inside the Codespaces environment, you’ll find all the files and instructions you need. Follow the steps outlined in the README file to complete your assignment.

1. **Read the Instructions**: Carefully go through the README file to understand the tasks you need to complete.
2. **Edit the Code**: Make the necessary changes to the code files as instructed.
3. **Run and Test Your Code**: Use the terminal and editor within Codespaces to run your code and make sure everything works as expected.

   > **Hint**: If you’re stuck, try reviewing the README file again or refer to any resources provided by your instructor.

### Step 4: Submit Your Work via Pull Request

Once you’ve completed the assignment, it’s time to submit your work. You’ll do this by creating a pull request, which is a way to propose your changes to the original repository.

1. **Commit Your Changes**:
   - Save your work by committing your changes. In Codespaces, go to the Source Control panel, write a commit message, and click "Commit" to save your changes.
2. **Push to Your Fork**:
   - After committing, click "Push" to upload your changes to your forked repository on GitHub.
3. **Create a Pull Request**:
   - Go back to your GitHub repository, and you’ll see an option to “Compare & pull request.” Click it to start your pull request.
   - Include your name in the pull request description so your instructor knows who submitted it.
4. **Submit the Pull Request**:
   - Click "Create pull request" to submit your work for review. Your instructor will be notified and can review your work.

And that’s it! You’ve now completed your first lab assignment using GitHub and Codespaces. Well done!

#### Additional Steps

1. Open the terminal in Codespaces.
2. Run the following commands to install dependencies and start the development server:

    ```sh
    npm install
    npm run dev
    ```

3. You can now view the project in the browser by clicking the "Application" port in the Ports panel.

Follow the instructions in the previous sections to complete the lab.