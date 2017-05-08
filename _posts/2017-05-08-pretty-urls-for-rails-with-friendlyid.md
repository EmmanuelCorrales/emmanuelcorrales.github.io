---
layout: post
title:  "Pretty URLs for Rails with FriendlyID"
date:   2017-05-08 07:37:33 +0800
categories: ruby rails friendly_id
tags: [ ruby, rails, friendly_id ]
---
<p>Rails by default uses a numeric id as a parameter for its url. Sometimes when
building a website such as this blog, instead of "emmmanuelcorrales.com/posts/4"
we want our url to be like "emmmanuelcorrales.com/posts/pretty-urls-for-rails-with-friendlyid".
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

<p>Add this line to your Gemfile.</p>
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

<p>On your PostController.rb instead of accessing the post object like the code below</p>

{% highlight ruby %}
# app/controllers/post_controller.rb
def set_post
  @post = Post.find(params[:id])
end
{% endhighlight %}

<p>use this</p>

{% highlight ruby %}
# app/controllers/post_controller.rb
def set_post
  @post = Post.friendly.find(params[:id])
end
{% endhighlight %}

<p>The app is finished. Execute the code below to run the app.</p>

{% highlight shell %}
rails s
{% endhighlight %}

<p>Go to <a href="http://localhost:3000/posts">http://localhost:3000/posts</a>
You can now set your custom slug whenever you edit your post. Happy coding!</p>
