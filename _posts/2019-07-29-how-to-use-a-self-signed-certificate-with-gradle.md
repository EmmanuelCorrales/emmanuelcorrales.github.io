---
layout: post
title:  "How to use a self signed certificate with Gradle"
date:   2019-07-29 08:30:00 +0800
categories:  java gradle  ssl https openssl
tags: [ java, gradle, ssl, https, openssl ]
---

Lately I've been getting a lot of issues regarding self-signed certificates with
Java apps using Gradle as their build tool. The issues were mostly errors like
*"Exception in thread "main" javax.net.ssl.SSLHandshakeException"* or packages
that can't be downloaded because of an invalid certificate. This was caused by
Gradle not importing the corporate certificiate pre-installed on my client's
company issued laptop by default. This post is a guide about using a self signed
certificate with Gradle.

First find out where the certificate is stored. The certificate can be in any
directory but if using OpenSSL it is good practice to put the certificate
on its default directory. Depending on the operating system, the default
directory may vary.
```bash
$ openssl version -d
OPENSSLDIR: "/private/etc/ssl"
```
Export the self signed certificate to the Java keystore.
```
keytool -import -trustcacerts -alias root -file /private/etc/ssl/certs.pem -keystore $JAVA_HOME/jre/lib/security/cacerts -storepass changeit
```
The default trust store is located at *$JAVA_HOME/jre/lib/security/cacerts*.
On the project's root directory create or edit the *gradle.properties* file
and add the line below.
```
org.gradle.jvmargs=-Djavax.net.ssl.keyStore="$JAVA_HOME/jre/lib/security/cacerts" -Djavax.net.ssl.keyStoreType=KeychainStore -Djavax.net.ssl.keyStorePassword=changeit
```
If you have properly done the steps above then there should no longer be any
problem when running Gradle.
