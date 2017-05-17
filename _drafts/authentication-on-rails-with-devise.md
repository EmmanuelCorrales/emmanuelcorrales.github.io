---
layout: post
title:  "Authentication on Rails with Devise"
date:   2017-05-17 06:22:00 +0800
categories: ruby rails devise authentication
tags: [ ruby, rails, devise, authentication ]
---
<p>In this tutorial, I'll show you how to use the <a href="https://github.com/plataformatec/devise">Devise</a>
gem for user authentication.</p>

<h2>Demo app</h2>
<p>I'll demonstrate how to use the <a href="https://github.com/plataformatec/devise">Devise</a>
gem by creating a simple demo app. You can download the source code
<a href="https://github.com/EmmanuelCorrales/rails-friendly_id-example">here</a>.<p>
<p>Create a new rails project by executing the command below.</p>

{% highlight shell %}
rails new devise-demo
{% endhighlight %}

<p>Add this line to the Gemfile.</p>
{% highlight ruby %}
# Gemfile
gem 'devise'
{% endhighlight %}

<p>Install the devise gem by executing the commands below.</p>

{% highlight shell %}
bundle install
{% endhighlight %}

<p>Generate devise configuration files by executing the command below.</p>

{% highlight shell %}
rails generate devise:install
{% endhighlight %}

<p>This will generate two configuration files <b>config/initializers/devise.rb</b>
and <b>config/locales/devise.en.yml</b> and show you some instructions for configuring
the app with devise.</p>
<br/>

<h3>Configuring the mailer</h3>
<p>Setup the mailer for the user's registration/recovery confirmation.<p>

<p>Configure the mailer for development by adding the line below to
<b>config/environments/development.rb</b>.</p>

{% highlight ruby %}
# Add this line to config/environments/development.rb
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
{% endhighlight %}

<p>The code above configures the mailer only for the development environment. In production,
<b>host:</b> should be set to the actual host of your application. To configure the mailer
for production add the line below on your <b>config/environments/production.rb</b>.</p>

{% highlight ruby %}
# Add this line to config/environents/production.rb
config.action_mailer.default_url_options = { host: 'www.example.com' }
{% endhighlight %}
<br/>

<h3>Default url after authentication</h3>
<p>Ensure you have defined root_url to *something* in your <b>config/routes.rb</b>.
The root url is where the authenticated user will be redirected after logging in.
For example:</p>

{% highlight ruby %}
# config/routes.rb
root to: "home#index"
{% endhighlight %}

<p>Generate a user model by running a command below</p>

{% highlight shell %}
rails generate devise user
rake db:migrate
{% endhighlight %}
