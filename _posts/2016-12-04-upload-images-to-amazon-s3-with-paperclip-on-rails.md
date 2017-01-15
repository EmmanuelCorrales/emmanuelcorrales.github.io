---
layout: post
title:  "Upload images to Amazon S3 with Paperclip on Rails"
date:   2016-12-10 00:22:33 +0800
categories: ruby rails s3 amazon web services
tags: [ ruby, rails, s3, amazon web services, figaro, imagemagick, fog, paperclip ]
---
<p>Sometimes uploading files to your server's local filesystem is not enough due
to some constraints like a limited disk space and potential security issues caused
by allowing users to upload files to your server. Using third party services like
Amazon S3 for file storage is a good way to solve these issues. I decided to use the
<a href="https://github.com/thoughtbot/paperclip">Paperclip</a> gem to implement
this functionality on my Rails app.</p>

<h2>Setup Amazon S3</h2>
<p>This article does not cover setting up an Amazon S3 bucket. This article assumes that you have
already setup your Amazon S3 bucket and familiar with your AWS credentials. You should
know your Amazon S3 bucket's name, access id, secret access key and region.</p>
<br/>

<h2>Demo app</h2>
<p>I'll demonstrate how to upload files to S3 with
<a href="https://github.com/thoughtbot/paperclip">Paperclip</a>
on Rails by building a simple demo app. You can download the source code
<a href="https://github.com/EmmanuelCorrales/rails-paperclip-s3-example">here</a>.</p>

<p>Create a new rails project by executing the command below.</p>

{% highlight shell %}
rails new paperclip-s3-example
{% endhighlight %}

<br/>

<h3>Setup Figaro</h3>

<p>Use the gem <a href="https://github.com/laserlemon/figaro">Figaro</a> to
make it easy to securely configure the rails demo application.</p>
<p>Add this line to your Gemfile.</p>

{% highlight ruby %}
gem 'figaro'
{% endhighlight %}

<p>Install figaro by executing the commands below.</p>

{% highlight shell %}
bundle install
bundle exec figaro install
{% endhighlight %}

<p>This will generate an <b>application.yml</b> file at the config directory.
The file's content should look like the code below. Replace the value with your
Amazon S3 bucket's credentials.</p>

<b>config/application.yml</b>
{% highlight ruby %}
S3_BUCKET_NAME: "mybucketname"
AWS_ACCESS_KEY_ID: "AKI84JDHFYRKW80Q43RQ"
AWS_SECRET_ACCESS_KEY: "HJD348asd3dgdj3ysdjshHDSJ39DSH393D"
AWS_REGION: "ap-southeast-1"
{% endhighlight %}

<p>This file contains the credentials for your S3 bucket and shouldn't be added
to your git repository, fortunately figaro adds this file to your <b>.gitignore</b>
file upon installation. A good practice for using figaro is adding an
<b>application.yml.template</b> file to your repository's config directory so that
other users can just copy its contents when setting up their application.yml on
their new development environment.</p>

<b>config/application.yml.template</b>
{% highlight ruby %}
S3_BUCKET_NAME: ""
AWS_ACCESS_KEY_ID: ""
AWS_SECRET_ACCESS_KEY: ""
AWS_REGION: ""
{% endhighlight %}

<br/>
<h3>Configure Paperclip to use Amazon S3</h3>
<p>Use the official <a href="https://github.com/aws/aws-sdk-ruby">AWS</a> and
<a href="https://github.com/thoughtbot/paperclip">Paperclip</a> gems.</p>

<p>Add these lines to the Gemfile.</p>

{% highlight ruby %}
gem 'paperclip', '~> 5.0.0'
gem 'aws-sdk', '~> 2'
{% endhighlight %}

<p>Install these gems by executing the code below.</p>

{% highlight shell %}
bundle install
{% endhighlight %}

<p>Modify and add the code below to your <b>config/environments/development.rb</b> and
<b>config/environments/production.rb</b>.</p>

{% highlight ruby %}
config.paperclip_defaults = {
    storage: :s3,
    s3_credentials: {
      bucket: ENV.fetch('S3_BUCKET_NAME'),
      access_key_id: ENV.fetch('AWS_ACCESS_KEY_ID'),
      secret_access_key: ENV.fetch('AWS_SECRET_ACCESS_KEY'),
      s3_region: ENV.fetch('AWS_REGION'),
    }
}
{% endhighlight %}

<br/>

<h3>Product model</h3>

<p>Generate a Product model with an attribute "name"  by executing the command below.</p>

{% highlight shell %}
rails g model product name:string
{% endhighlight %}

<p>Add an image attachment attribute by execute the command below.</p>

{% highlight shell %}
rails g paperclip product image
{% endhighlight %}

<p>This will generate a migration like the code below.</p>

<b>db/migrate/20161203110020_add_attachment_image_to_products.rb</b>
{% highlight ruby %}
class AddAttachmentImageToProducts < ActiveRecord::Migration
  def self.up
    change_table :products do |t|
      t.attachment :image
    end
  end

  def self.down
    remove_attachment :products, :image
  end
end
{% endhighlight %}

<p>Execute the migration by running the code below.

{% highlight shell %}
rake db:migrate
{% endhighlight %}

<p>Modify the Product model to add attachment functionality. Use the Paperclip
helper method has_attached_file and a symbol with the desired name of the attachment.</p>

<b>app/models/product.rb</b>

{% highlight ruby %}
class Product < ActiveRecord::Base
  has_attached_file :image, styles: {
    thumb: '100x100>',
    preview: '300x225#',
    large: '600x600>'
  }
  validates_attachment_content_type :image, :content_type => /\Aimage\/.*\Z/
end
{% endhighlight %}

<br/>

<h3>Products controller</h3>

<p>Generate a controller for products.</p>

{% highlight shell %}
rails g controller products
{% endhighlight %}


<p>Modify the ProductsController to look like the code below.</p>

<b>app/controllers/products_controller.rb</b>
{% highlight ruby %}
class ProductsController < ApplicationController
  before_action :set_product, only: [:show, :edit, :update, :destroy]

  def index
    @products = Product.all
  end

  def show
  end

  def new
    @product = Product.new
  end

  def edit
  end

  def create
    @product = Product.new(product_params)
    if @product.save
      redirect_to @product, notice: 'Product was successfully created.'
    else
      render :new
    end
  end

  def update
    if @product.update(product_params)
      redirect_to @product, notice: 'Product was successfully updated.'
    else
      render :edit
    end
  end

  def destroy
    @product.destroy
    redirect_to products_url, notice: 'Product was successfully destroyed.'
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_product
      @product = Product.find(params[:id])
    end

    # Never trust parameters from the scary internet, only allow the white list through.
    def product_params
      params.require(:product).permit(:name, :image)
    end
end
{% endhighlight %}

<br/>

<h3>Diplaying the image on your views</h3>

<p>Create an index file for our homepage. Its content should look like the code
below and take note of the <b>product.image.url(:thumb)</b>. It is the
reference to the resized version of the image.</p>

<b>app/views/products/index.html.erb</b>

{% highlight rhtml %}
<p id="notice"><%= notice %></p>
<h1>Listing Products</h1>

<table>
  <thead>
    <tr>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <% @products.each do |product| %>
      <tr>
        <td><%= product.name %></td>
        <td><%=  image_tag product.image.url(:thumb) %></td>
        <td><%= link_to 'Show', product %></td>
        <td><%= link_to 'Edit', edit_product_path(product) %></td>
        <td><%= link_to 'Destroy', product, method: :delete, data: { confirm: 'Are you sure?' } %></td>
      </tr>
    <% end %>
  </tbody>
</table>

<br>

<%= link_to 'New Product', new_product_path %>
{% endhighlight %}

<p>Create a partial form for our <b>new</b> and <b>edit</b> actions. The <b>file_field</b> will
be used for selecting a file to upload. Take note of <b>:html => {:multipart => true}</b>,
this is necessary to upload the image in multiple chunks.</p>
<b>app/views/products/_form.html.erb</b>

{% highlight rhtml %}
<%= form_for(@product,:html => {:multipart => true}) do |f| %>
  <% if @product.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@product.errors.count, "error") %> prohibited this product from being saved:</h2>

      <ul>
      <% @product.errors.full_messages.each do |message| %>
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
    <%= f.file_field :image %>
  </p>

  <div class="actions">
    <%= f.submit %>
  </div>
<% end %>
{% endhighlight %}


<p>Create our view for our <b>show</b> action. Notice that I use <b>photo.image.url</b>
instead of <b>photo.image.url(:thumb)</b> like on our index page. This shows the original
size of the uploaded.</p>
<b>app/views/products/show.html.erb</b>

{% highlight rhtml %}
<p id="notice"><%= notice %></p>
<td><%= @product.name %></td>
<td><%=  image_tag @product.image.url %></td>
<%= link_to 'Edit', edit_product_path(@product) %> |
<%= link_to 'Back', products_path %>
{% endhighlight %}

<b>app/views/photos/new.html.erb</b>

{% highlight rhtml %}
<h1>New Product</h1>
<%= render 'form' %>
<%= link_to 'Back', products_path %>
{% endhighlight %}

<b>app/views/photos/edit.html.erb</b>

{% highlight rhtml %}
<h1>Editing Product</h1>
<%= render 'form' %>
<%= link_to 'Show', @product %> |
<%= link_to 'Back', products_path %>
{% endhighlight %}

<br/>

<p>Make sure we have configured our routes properly.</p>
<b>config/routes.rb</b>

{% highlight ruby %}
Rails.application.routes.draw do
  resources :products
end
{% endhighlight %}

<p>The photos app is finished. Execute the code below.</p>

{% highlight shell %}
rails s
{% endhighlight %}

<p>Go to <a href="http://localhost:3000/photos">http://localhost:3000/photos</a>
 and see the rails photo app work. Happy coding!</p>
