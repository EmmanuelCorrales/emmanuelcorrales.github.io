---
layout: post
title:  "Pretty URLs for Rails with FriendlyID"
date:   2017-05-08 07:37:33 +0800
categories: ruby rails friendly_id
tags: [ ruby, rails, friendly_id ]
---
<p>Rails by default uses a numeric id as a parameter for its URL. Sometimes when
building a website such as this blog, instead of 'blog.emmmanuelcorrales.com/posts/4'
we want the url to be like 'blog.emmmanuelcorrales.com/posts/pretty-urls-for-rails-with-friendlyid'.
We can achieve this by using the <a href="https://github.com/norman/friendly_id">FriendlyID</a>
gem. In this tutorial I'll show you how to use <a href="https://github.com/norman/friendly_id">FriendlyID</a>.</p>


<h2>Blog app</h2>
<p>I'll demonstrate how to use <a href="https://github.com/norman/friendly_id">FriendlyID</a>
gem by creating a simple blog app. You can download the source code
<a href="https://github.com/EmmanuelCorrales/rails-friendly_id-example">here</a>.<p>
<p>Create a new rails project by executing the command below.</p>

{% highlight shell %}
rails new blog
{% endhighlight %}

<p>Add this line to the Gemfile.</p>
{% highlight ruby %}
# Gemfile
gem 'friendly_id', '~> 5.1.0'
{% endhighlight %}

<p>Install friendly_id by executing the commands below.</p>

{% highlight shell %}
bundle install
{% endhighlight %}

<p>Run the command below to generate a migration that will create a table for the slugs.</p>
{% highlight shell %}
rails generate friendly_id
{% endhighlight %}

<p>Generate a scaffold for our post with attributes "title", "description" and "slug".</p>
{% highlight shell %}
rails generate scaffold post title:string description:text slug:string:uniq
rake db:migrate
{% endhighlight %}

<p>Modify your Post model to look like the code below.</p>
{% highlight ruby %}
# app/models/post.rb
class Post < ApplicationRecord
  extend FriendlyId
  friendly_id :title, use: :slugged
end
{% endhighlight %}

<p>On your PostController.rb instead of accessing the post object like the code below.</p>

{% highlight ruby %}
# app/controllers/post_controller.rb
def set_post
  @post = Post.find(params[:id])
end
{% endhighlight %}

<p>Access it using the friendly method like the code below.</p>

{% highlight ruby %}
# app/controllers/post_controller.rb
def set_post
  @post = Post.friendly.find(params[:id])
end
{% endhighlight %}

<p>The app is finished. Run the app.</p>

{% highlight shell %}
rails serve
{% endhighlight %}

<p>Go to <a href="http://localhost:3000/posts">http://localhost:3000/posts</a>
You can now set your custom slug whenever you edit your post.</p>
<br/>

<h3>Automatically update the slug based on the title</h3>

<p>Personally I want the slug to be automatically updated based on the title so
whenever I edit the title, I don't have to manually edit the slug. The app
already uses the title to generate the slug when creating
a new post but updating it doesn't change the slug.</p>

<p>We need to omit the block of code from the partial form where the slug is
edited because its sets the value of the slug. The slug must be nil for it to be
regenerated.</p>

{% highlight rhtml %}
<%-# Remove this block of code from app/views/posts/_form.html.erb -%>
<div class="field">
   <%= f.label :slug %>
   <%= f.text_field :slug %>
</div>
{% endhighlight %}

<p>The partial form with the omitted code should be similar to the code below.</p>

{% highlight rhtml %}
<%-# app/views/posts/_form.html.erb -%>
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

  <div class="field">
    <%= f.label :title %>
    <%= f.text_field :title %>
  </div>

  <div class="field">
    <%= f.label :content %>
    <%= f.text_area :content %>
  </div>

  <div class="actions">
    <%= f.submit %>
  </div>
<% end %>
{% endhighlight %}

<p>Set the post's slug as nil before it is updated to generate the new slug on update.</p>

{% highlight ruby %}
def update
    @post.slug = nil
    @post.update(post_params)
end
{% endhighlight %}

<p>Now whenever you edit the title of the post the slug will change.</p>
