---
layout: post
title:  "AWS IAM Roles vs Acces keys"
date:   2018-02-24 08:30:00 +0800
categories: aws iam
tags: [ aws, roles]
---

<p>I've been studying about IAM roles this weekend and I learned the
 difference of using roles and access keys. I created a blog post
 before about uploading images to Amazon S3 with Rails.
 On that post I was using AWS access keys to upload files. I remembered
 that I developed an application with a similar configuration and deployed
 it to an EC2 instance before so I tested it with 'roles' instead of 'access keys'.</p>

<h3>What are IAM Roles?</h3>
<p>An IAM Role is an identity. An IAM role has permission policies.
 Permission policies determine what an identity can and cannot
 do(ex. Creating/Deleting an S3 bucket). You can add/remove one or
 more policies at any time. Roles are meant to be assigned to multiple usersi or services.
 (ex. 20 EC2 instances running on the same role.)
 A role has no credentials like a password or access key.</p>

<h3>What are Credentials?</h3>
<p>Credentials are handed out to users when they are assigned to a role.
 It removes the need to rotate and assign keys.
 It is useful for giving access to users, applications and services.</p>

<h3>Development Scenarios</h3>

<p>EC2 intances with applications on them. The application need to access services like S3.
 Instead of manually assigning credentials and rotating them, use IAM Roles.

<p>Mobile applications that access services like S3 require access keys.
 This is not secure because it can be decompiled and extracted.</p>
