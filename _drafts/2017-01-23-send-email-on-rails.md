---
layout: post
title:  "Send an email on Rails"
date:   2017-01-23 17:06:33 +0800
categories: ruby rails gmail
tags: [ ruby, rails, figaro, action mailer ]
---
<p>Send an email on Rails. I'll demonstrate how to send an email on Rails by building
an app that sends a welcome email to the user after it was created.</p>

<p>Create a new app by executing the code below.<p>

{% highlight shell %}
rails new welcome-email-example
{% endhighlight %}

<p>Generate a user model with attributes name and email then migrate by executing
the commands below.</p>

{% highlight shell %}
rails g model user name email
rake db:migrate
{% endhighlight %}

<p>Generate a user mailer.</p>

{% highlight shell %}
rails g mailer UserMailer
{% endhighlight %}

<p>Setup the default values of our mailer. The code below sets the default
sender of email. We also setup the default layout for our email.</p>

<b>app/mailers/application_mailer.rb</b>
{% highlight ruby %}
class ApplicationMailer < ActionMailer::Base
  default from: "from@example.com"
  layout 'mailer'
end
{% endhighlight %}

<p>We create a mailer action that accepts a user as a parameter and call it
<b>welcome_email</b>.</p>

<b>app/mailers/user_mailer.rb</b>
{% highlight ruby %}
class UserMailer < ApplicationMailer

  def welcome_email(user)
    @user = user
    mail(to: @user.email, subject: 'Welcome!')
  end

end
{% endhighlight %}

<br/>
<h3>Mailer Views</h3>
<p>Mailer is just like a controller it has its own views. It supports html and plain text.</p>
<b>app/views/user_mailer/welcome_email.html.erb</b>

{% highlight rhtml %}
<!DOCTYPE html>
<html>
  <head>
    <meta content='text/html; charset=UTF-8' http-equiv='Content-Type' />
  </head>
  <body>
    <h1>Welcome to example.com, <%= @user.name %></h1>
    <p>
      You have successfully created a new user.
    </p>
  </body>
</html>
{% endhighlight %}

<b>app/views/user_mailer/welcome_email.text.erb</b>
{% highlight rhtml %}
Welcome , <%= @user.name %>

You have successfully created a new user.
{% endhighlight %}

<br/>
<h3>Controller</h3>

<p>Generate a user controller.</p>

{% highlight shell %}
rails g controller users
{% endhighlight %}

<b>app/controllers/users_controller.rb</b>

{% highlight ruby %}
class UserController < ApplicationController

  def index
    @user = User.all
  end

  def new
    @user = User.new
  end

  def create
    @user = User.new(user_params)
    if @user.save
      UserMailer.welcome_email(@user).deliver_now
      redirect_to @user, notice: 'User was successfully created.'
    else
      render new
    end
  end

  private

    def user_params
      params.require(:user).permit(:name, :email)
    end

end
{% endhighlight %}

<p>The app would send an email after creating a new user.</p>

<br/>
<h3>Views</h3>

<b>app/views/user/index.html.erb</b>

{% highlight rhtml %}
<p id="notice"><%= notice %></p>
<h1>Listing Users</h1>

<table>
  <thead>
    <tr>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <% @users.each do |user| %>
      <tr>
        <td><%= user.name %></td>
        <td><%= user.email %></td>
      </tr>
    <% end %>
  </tbody>
</table>

<br>

<%= link_to 'New User', new_user_path %>
{% endhighlight %}

<b>app/views/user/new.html.erb</b>

{% highlight rhtml %}
<h1>New User</h1>
<%= render 'form' %>
<%= link_to 'Back', users_path %>
{% endhighlight %}

<b>app/views/user/_form.html.erb</b>

{% highlight rhtml %}
<%= form_for(@user,:html => {:multipart => true}) do |f| %>
  <% if @user.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@user.errors.count, "error") %> prohibited this user from being saved:</h2>

      <ul>
      <% @user.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

  <p>
    <%= f.label :name %><br/>
    <%= f.text_field :name %>
  </p>
  <p>
    <%= f.label :email %><br/>
    <%= f.text_field :email %>
  </p>
  <div class="actions">
    <%= f.submit %>
  </div>
<% end %>
{% endhighlight %}
