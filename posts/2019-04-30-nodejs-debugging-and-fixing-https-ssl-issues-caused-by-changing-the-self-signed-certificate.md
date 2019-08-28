---
layout: post
title:  "NodeJS: Debugging and fixing HTTPS/SSL issues caused by changing the self signed certificate."
date:   2019-04-30 08:30:00 +0800
categories:  nodejs javascript ssl
tags: [ nodejs, javascript, ssl ]
---
This past month I've been bombarded with lots of issues regarding HTTPS/SSL.
The internal Certificate Authority of the company I'm servicing recently changed
their root certicate. This caused the services that communicate with other
services in their network through HTTPS/SSL to break and receive errors. I've
been debugging a NodeJS app that is being plagued by these issues. In this post
I'll show you the steps I took to fix this.

![alt text](https://images.emcorrales.com/NodeJS-SSL-old-architecture.png
"The NodeJs app that depend on a third party app broke after changing certificate")

The image above shows that the NodeJS app uses an API on their internal network.
The NodeJS app is the client of the third party API and is having trouble getting
a response through HTTPS even though the admin has installed the latest
certificate on every server's trust store and is ready for use.

### Checking if the certificate is installed on the server.
The first thing I did is to check if the new certificate is installed on the
server of the client app. I looked for the directory where the certificates are
stored:
```bash
$ openssl version -d
OPENSSLDIR: "/usr/lib/ssl"
```
then I looked for the certifcate in that directory. Below is the command I used
to look for a certificate called *example.pem*.
```bash
ls /usr/lib/ssl/certs | grep example.pem
```

### Verifying the connection to the third party API.
I needed to figure out if there is a problem with the new certificate. The first
thing that comes to my mind is to check if I can establish a HTTPS connection
with the third party API so I executed this:
```bash
openssl s_client -connect api.example.com:443
```
Once I verified the HTTPS/SSL connection with the third party API, I proceeded
to running the client app.

### Running the client app.
The Node app I was debugging is too big and I just need to test the part that
uses the API so I created a simple app that uses the same API. This will
serve as proxy. Below are the contents of the file *client.js*. A simple
NodeJS app that uses the third party API.
```javascript
// client.js
const https = require('https');

https.get("https://api.example.com", (res) => {
  console.log(res.statusCode);
  if (res.statusCode !== 200) {
    throw new Error(`Expected 200, got ${res.statusCode}`);
  }

  res.on('data', data => process.stdout.write(data));
});
```
When I run the app I  get error messages like these:
```bash
$ node client.js
Error: unable to verify the first certificate
    at TLSSocket.onConnectSecure (_tls_wrap.js:1036:34)
    at TLSSocket.emit (events.js:159:13)
    at TLSSocket._finishInit (_tls_wrap.js:637:8)
```
The app is having verification errors with the API's certificate because Node
doesn't load the corresponding certificate that can verify it when creating a
 HTTPS request. The new certificate is installed on the server's trust store but
Node doesn't use it because it already has a default list of CA's built into its
[source](https://github.com/joyent/node/blob/master/src/node_root_certs.h).
Thus, while it is true that it doesn't do a lookup on the system hosted CA's
and that there is no "store" per se, there is a default list of CA's that it
accepts.

In order to use the new certificate, the app must be configured to load the CA
upon deployment by passing the environment variable **NODE_EXTRA_CA_CERTS** like
this:
```bash
NODE_EXTRA_CA_CERTS=/usr/lib/ssl/certs node client.js Hello, world
```
This time the app will no longer show an error because it knows where to find
the new certificate.
