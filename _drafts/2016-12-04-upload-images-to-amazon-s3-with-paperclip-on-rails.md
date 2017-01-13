---
layout: post
title:  "Upload images to Amazon S3 with Paperclip on Rails"
date:   2016-12-10 00:22:33 +0800
categories: ruby rails s3 amazon web services
tags: [ ruby, rails, s3, amazon web services, figaro, imagemagick, fog, paperclip ]
---
<p>Sometimes uploading files to your server's local filesystem is not enough due to some constraints like a limited disk space or bandwidth and potential security issues caused by allowing users to upload files to your server. Most developers prefer to upload their files to Amazon S3.</p>

<h2>Setup Amazon S3</h2>
<p>This article does not cover setting up an Amazon S3 bucket. This article assumes that you have
already setup your Amazon S3 bucket and familiar with your AWS credentials. You should
know your bucket's name, host name, AWS access id, AWS secret access key and AWS region.</p>
<br/>

<h2>Products app</h2>
<p>I'll demonstrate how to upload files to S3 with
<a href="https://github.com/thoughtbot/paperclip">Paperclip</a>
on Rails by building a simple photos app. You can download the source code
<a href="https://github.com/EmmanuelCorrales/rails-paperclip-s3-example">here</a>.</p>

<p>Create a new rails project by executing the command below.</p>

{% highlight shell %}
rails new paperclip-s3-example
{% endhighlight %}

<p>Generate and migrate a Product model with an attribute "name"  by
executing the command below.</p>

{% highlight shell %}
rails g model product name:string
rake db:migrate
{% endhighlight %}

<br/>

<h3>Setup Figaro</h3>

<p>Use the gem <a href="https://github.com/laserlemon/figaro">Figaro</a> to
make it easy to securely configure Rails applications.</p>
<p>Add figaro gem to your Gemfile.</p>

{% highlight ruby %}
gem 'figaro'
{% endhighlight %}

<p>Install figaro.</p>

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
<h3>Configure Paperclip</h3>

<p>Add these lines to the Gemfile.</p>

{% highlight ruby %}
gem "paperclip", "~> 5.0.0"
gem 'aws-sdk', '~> 2.3'
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
      s3_host_name: ENV.fetch('S3_HOST_NAME'),
    }
}
{% endhighlight %}

<p>Create a migration that adds an image attribute to the product model.</p>

{% highlight shell %}
rails g migration add_image_to_products
{% endhighlight %}

<p>Modify the generated migration to look like the code below.</p>
{% highlight ruby %}
class AddImageToProducts < ActiveRecord::Migration
  def self.up
    add_attachment :products, :image
  end

  def self.down
    remove_attachment :products, :image
  end
end
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

<h3>Controller and views</h3>

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

<p>The photos app is now finished. Now execute the code below.</p>

{% highlight shell %}
rails s
{% endhighlight %}

<p>Now go to <a href="http://localhost:3000/photos">http://localhost:3000/photos</a>
 and see the rails photo app work. Happy coding!</p>
