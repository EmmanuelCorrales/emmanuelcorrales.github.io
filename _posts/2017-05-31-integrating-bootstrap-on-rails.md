---
layout: post
title:  "Integrating Bootstrap on Rails"
date:   2017-05-31 08:30:00 +0800
categories: ruby rails bootstrap
tags: [ ruby, rails, bootstrap ]
---
<p>Bootstrap is the most popular HTML, CSS, and JavaScript framework for developing
responsive, mobile-first web sites. In this tutorial, I'll show you how to integrate
Bootstrap on your Rails application. This tutorial is not for learning Bootstrap,
if you want to learn Bootstrap, I recommend you visit this
<a target="_blank" href="https://www.w3schools.com/bootstrap/default.asp">link</a>.</p>

<h2>Demo app</h2>

<p>I'll create a new rails app to demonstrate how to integrate Bootstrap on Rails.
The source code can be downloaded
<a target="_blank" href="https://github.com/EmmanuelCorrales/rails-bootstrap-example">here</a>.
If you already have an existing application, you may want to skip some sections and
jump straight to <a href="#integrating-bootstrap">integrating bootstrap</a>.</p>

<p>Create a new rails app.</p>

{% highlight shell %}
rails new bootstrap-rails-example
{% endhighlight %}

<p>Install the gems with the command below.</p>

{% highlight shell %}
bundle install
{% endhighlight %}

<p>Scaffold a <b>post</b> with attributes <b>name</b> and <b>description</b> and
run a migration.</p>

{% highlight shell %}
rails g scaffold post title:string description:string
rake db:migrate
{% endhighlight %}

<p>Run the rails app with command below.</p>

{% highlight shell %}
rails serve
{% endhighlight %}

<p>Visit <a href="http://localhost:3000/posts/new" target="_blank">localhost:3000/posts/new</a>
 on your browser to see how your rails app looks without Bootstrap.</p>

<br/>

<h3>Apply Bootstrap on views</h3>

<p>To keep the scope of this tutorial small, I'll just demonstrate using Bootstrap
classes on the form. Below is an example of a form using Bootstrap.</p>

{% highlight rhtml linenos %}
<!-- app/views/posts/_form.html.erb -->
<div class="container">
  <%= form_for(post) do |f| %>
    <% if post.errors.any? %>
      <div id="error_explanation">
        <h2><%= pluralize(post.errors.count, "error") %> prohibited this post from being saved:</h2>

        <ul>
        <% post.errors.full_messages.each do |message| %>
          <li><%= message %></li>
        <% end %>
        </ul>
      </div>
    <% end %>

    <div class="form-group">
      <%= f.label :title %>
      <%= f.text_field :title, class:"form-control", type:"text" %>
    </div>

    <div class="form-group">
      <%= f.label :description %>
      <%= f.text_field :description, class:"form-control", type:"text" %>
    </div>

    <div class="actions">
      <%= f.submit "Submit", class:"btn btn-primary btn-block" %>
    </div>
  <% end %>
</div>
{% endhighlight %}

<p>The form is wrapped inside the div tag at line 2 with a <b>container</b> class which is
a class from Bootstrap. On line 16 and 21 it is using <b>form-group</b> instead
of the default <b>field</b>. On line 18 and 23 the text_field uses <b>form-control</b>
class. On line 27 the button uses <b>btn btn-primary btn-block</b> class. I've
already added the classes for Bootstrap but it still won't look like the way I want
it because I still haven't integrated Bootstrap.</p>

<br/>
<h3 id="integrating-bootstrap">Integrating Bootstrap.</h3>

<p>Add these lines to the Gemfile.</p>

{% highlight ruby %}
# Gemfile
gem 'bootstrap-sass', '~> 3.3.6'
gem 'autoprefixer-rails'
{% endhighlight %}

<p>Install these gems by executing the command below.</p>

{% highlight shell %}
bundle install
{% endhighlight %}

<p>Add these lines to the <b>application.scss</b> and make sure <b>bootstrap-sprockets</b>
is imported before <b>bootstrap</b> for the icon fonts to work. You have to rename <b>application.css</b> to <b>application.scss</b></p>

{% highlight scss %}
// app/assets/stylesheets/application.scss
@import "bootstrap-sprockets";
@import "bootstrap";
{% endhighlight %}

<p>Add this line to the <b>application.js</b> and make sure that <b>//= require_tree .</b>
is the last to be required.</p>

{% highlight javascript %}
//*require bootstrap-sprockets
{% endhighlight %}

<p>The contents of the <b>application.js</b> should be similar to the code below.</p>
{% highlight javascript %}
//= require jquery
//= require jquery_ujs
//= require turbolinks
//= require bootstrap-sprockets
//= require_tree .
{% endhighlight %}

<p>Visit <a href="http://localhost:3000/posts/new" target="_blank">localhost:3000/posts/new</a>
 on your browser to see how your rails app looks with Bootstrap.</p>
