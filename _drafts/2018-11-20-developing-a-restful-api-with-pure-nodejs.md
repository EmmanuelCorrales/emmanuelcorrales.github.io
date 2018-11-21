---
layout: post
title:  "Developing a RESTful API with pure NodeJS"
categories: nodejs javascript es6 api
date: 2018-11-20 08:30:00 +0800
tags: [ nodejs, javascript, es6, api ]
---
I've been studying NodeJS this past month to prepare for my next project at
work. This project may require the knowledge of developing with pure Node based
on me and my colleague's speculations so I decided to try and implement a simple
RESTful API without any other external libraries and using only Node's built-in
modules. In this article, I'll demonstrate how to implement it.

This tutorial requires that you are knowledgeable at Javascript ES6 syntax and
must have the LTS version of Node(v8.10.0) already installed.

## Create a new project
Create a new directory called **hello-node** with an **index.js** file inside
for our new project by running the commands below.
```bash
mkdir hello-node
cd hello-node
touch index.js
```
The **index.js** file will be the entry point of the application.

Edit the contents of the **index.js** file to look like the code below.
```javascript
const http = require('http');

const listener = (req, res) => {
  res.end("Hello NodeJS!");
  console.log("A request has been made!");
};

const server = http.createServer(listener);

server.listen(3000, () => {
  console.log("Server is listening at port 3000.");
});
```
In the code above, we initially imported Node's "**http**" module for
instantiating an HTTP server which would listen and respond to requests. The
variable **listener** is assigned a function that will be called whenever the
server receives a request. It is also used as the argument to instantiate an
HTTP server. The server would then listen for HTTP request at port 3000.

Lets run the app by executing the command below.
```bash
node index.js
```
Then lets test if the app can respond to HTTP requests. Open another terminal
and run the command below or just open <http://localhost:3000> on your browser.
```bash
curl localhost:3000
```
You should see the text "**Hello NodeJS!**" if the app has responded properly.
If you look at the terminal session where we run the Node app, the text
"**A request has been made!**" should be logged.

## Disecting the request
We will be disecting the components of the HTTP request. We need to find out
the method, headers, path, query string and payload.
We are going to disect then log the details of the request.
```javascript
  const { method, headers } = req;
  const { pathname, query } = url.parse(req.url, true);
```

```javascript
const http = require('http');
const url = require('url');
const { StringDecoder } = require('string_decoder');

const server = http.createServer((req, res) => {
  const { method, headers } = req;
  const { pathname, query } = url.parse(req.url, true);

  const headerString = Object.keys(headers).reduce((acc, key) => {
    return acc + `\n  ${ key }: ${ headers[key] }`;
  }, 'HEADERS:');

  const queryString = Object.keys(query).reduce((acc, key) => {
    return acc + `\n  ${ key }: ${ query[key] }`;
  }, 'QUERY STRINGS:');

  const decoder = new StringDecoder('utf-8');
  let payload = '';

  req.on('data', (data) => {
    payload += decoder.write(data);
  });

  req.on('end', () => {
    payload += decoder.end();
    console.log(`${ method } ${ pathname }\n${ headerString }\n${ queryString }`
        + `\nPAYLOAD:\n  ${ payload }`);
    res.end("Hello NodeJS!\n");
  });

});

server.listen(3000, () => {
  console.log("Server is listening at port 3000.");
});
```
