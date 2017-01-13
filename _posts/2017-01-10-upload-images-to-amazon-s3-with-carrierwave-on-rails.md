---
layout: post
title:  "Upload images to Amazon S3 with Carrierwave on Rails."
date:   2017-01-10 00:22:33 +0800
categories: ruby rails s3 amazon web services
tags: [ ruby, rails, s3, amazon web services, figaro, carrierwave, mini_magick, fog, paperclip ]
---
<p>Sometimes uploading files to your server's local filesystem is not enough due
to some constraints like a limited disk space or bandwidth and potential security
issues caused by allowing users to upload files to your server.
Most developers prefer to upload their files to Amazon S3. While searching for
ways to upload images on Rails, I stumbled upon two gems that simplifies this task,
<a href="https://github.com/thoughtbot/paperclip">Paperclip</a> and
<a href="https://github.com/carrierwaveuploader/carrierwave">Carrierwave</a>.
I tried Paperclip first because there's a good documentation on how to use it on
the Heroku website, however a senior colleague of mine recommended
<a href="https://github.com/carrierwaveuploader/carrierwave">Carrierwave</a>
because it is much simpler and cleaner to implement.</p>

<h2>Setup Amazon S3</h2>
<p>This article does not cover setting up an Amazon S3 bucket. This article assumes that you have
already setup your Amazon S3 bucket and familiar with your AWS credentials. You should
know your bucket's name, host name, AWS access id, AWS secret access key and AWS region.</p>
<br/>

<h2>Photos app</h2>
<p>I'll demonstrate how to upload files to S3 with
<a href="https://github.com/carrierwaveuploader/carrierwave">Carrierwave</a>
on Rails by building a simple photos app. You can download the source code
<a href="https://github.com/EmmanuelCorrales/rails-carrierwave-s3-example">here</a>.</p>

<p>Create a new rails project by executing the command below.</p>

{% highlight shell %}
rails new photos
{% endhighlight %}

<p>Generate and migrate a Photo model with attributes "name" and "image" by
executing the command below.</p>

{% highlight shell %}
rails g model photo name:string image:string
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
<h3>Setup Carrierwave</h3>

<p>Add these lines to the Gemfile.</p>

{% highlight ruby %}
gem 'carrierwave'
gem 'mini_magick'
gem 'fog'
{% endhighlight %}

<p><a href="https://github.com/carrierwaveuploader/carrierwave">Carrierwave</a>
is a gem for uploading files.
<a href="https://github.com/minimagick/minimagick">MiniMagick</a> is a gem for
resizing images and <a href="https://github.com/fog/fog-aws">Fog::AWS</a> is a
gem that adds support for Amazon Web Services. Install these gems by executing
the command below.</p>

{% highlight shell %}
bundle install
{% endhighlight %}

<p>Generate an image uploader by executing the command below.</p>
{% highlight shell %}
rails g uploader image
{% endhighlight %}

<p>Executing the command above generates an ImageUploader class at
<b>app/uploaders/image_uploader.rb</b></p>

{% highlight ruby %}
class ImageUploader < CarrierWave::Uploader::Base

  include CarrierWave::MiniMagick

  storage :fog

  def store_dir
    "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
  end

  version :thumb do
    process :resize_to_fit => [950, 850]
  end

end
{% endhighlight %}

<p><a href="https://github.com/carrierwaveuploader/carrierwave">Carrierwave</a>
supports the gems <a href="https://github.com/minimagick/minimagick">MiniMagick</a>
and <a href="https://github.com/rmagick/rmagick">RMagick</a> for photo resizing.
In this case I chose to use <a href="https://github.com/minimagick/minimagick">MiniMagick</a>
because it is more lightweight compared to <a href="https://github.com/rmagick/rmagick">RMagick</a>.
Set fog as the storage option instead of file, because we are gonna use Amazon S3 for storage.
The method <b>stor_dir</b> sets where the file should be uploaded.</p>

<br/>
<p>Mount the image uploader on the Photo model.</p>
<b>app/models/photo.rb</b>

{% highlight ruby %}
class Photo < ActiveRecord::Base
  mount_uploader :image, ImageUploader
end
{% endhighlight %}

<br/>
<p>Configure <a href="https://github.com/carrierwaveuploader/carrierwave">Carrierwave</a>
to use S3 credentials and stop it from using SSL when accessing the bucket.<p>

<b>config/initializers/carrierwave.rb</b>

{% highlight ruby %}
CarrierWave.configure do |config|
  config.fog_credentials = {
      :provider               => 'AWS',
      :aws_access_key_id      => ENV['AWS_ACCESS_KEY_ID'],
      :aws_secret_access_key  => ENV['AWS_SECRET_ACCESS_KEY'],
      :region                 => ENV['AWS_REGION']
  }
  config.fog_directory  = ENV['S3_BUCKET_NAME']
  config.fog_use_ssl_for_aws = false
end
{% endhighlight %}

<br/>
<br/>
<h3>Controller and views</h3>
<p>Create a Photo controller by executing the command below.</p>

{% highlight shell %}
rails g controller photos
{% endhighlight %}

<p>Modify the PhotosController to
look like the code below.</p>

<b>app/controllers/photos_controller.rb</b>
{% highlight ruby %}
class PhotosController < ApplicationController
  before_action :set_photo, only: [:show, :edit, :update, :destroy]

  def index
    @photos = Photo.all
  end

  def show
  end

  def new
    @photo = Photo.new
  end

  def edit
  end

  def create
    @photo = Photo.new(photo_params)
    if @photo.save
      redirect_to @photo, notice: 'Photo was successfully created.'
    else
      render :new
    end
  end

  def update
    if @photo.update(photo_params)
      redirect_to @photo, notice: 'Photo was successfully updated.'
    else
      render :edit
    end
  end

  def destroy
    @photo.destroy
    redirect_to photos_url, notice: 'Photo was successfully destroyed.'
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_photo
      @photo = Photo.find(params[:id])
    end

    # Never trust parameters from the scary internet, only allow the white list through.
    def photo_params
      params.require(:photo).permit(:name, :image)
    end
end
{% endhighlight %}

<p>Create an index file for our homepage. Its content should look like the code
below and take note of the <b>photo.image.thumb</b>. It is the
reference to the resized version of the image.</p>

<b>app/views/photos/index.html.erb</b>

{% highlight rhtml %}
<p id="notice"><%= notice %></p>
<h1>Listing Photos</h1>

<table>
  <thead>
    <tr>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <% @photos.each do |photo| %>
      <tr>
        <td><%= photo.name %></td>
        <td><%=  image_tag photo.image.thumb %></td>
        <td><%= link_to 'Show', photo %></td>
        <td><%= link_to 'Edit', edit_photo_path(photo) %></td>
        <td><%= link_to 'Destroy', photo, method: :delete, data: { confirm: 'Are you sure?' } %></td>
      </tr>
    <% end %>
  </tbody>
</table>

<br>

<%= link_to 'New Photo', new_photo_path %>
{% endhighlight %}

<p>Create a partial form for our <b>new</b> and <b>edit</b> actions. The <b>file_field</b> will
be used for selecting a file to upload. Take note of <b>:html => {:multipart => true}</b>,
this is necessary to upload the image in multiple chunks.</p>
<b>app/views/photos/_form.html.erb</b>

{% highlight rhtml %}
<%= form_for(@photo,:html => {:multipart => true}) do |f| %>
  <% if @photo.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@photo.errors.count, "error") %> prohibited this photo from being saved:</h2>

      <ul>
      <% @photo.errors.full_messages.each do |message| %>
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


<p>Create our view for our <b>show</b> action. Notice that I use <b>photo.image</b>
instead of <b>photo.image.thumb</b> like on our index page. This shows the original
size of the uploaded.</p>
<b>app/views/photos/show.html.erb</b>

{% highlight rhtml %}
<p id="notice"><%= notice %></p>
<td><%= @photo.name %></td>
<td><%=  image_tag @photo.image %></td>
<%= link_to 'Edit', edit_photo_path(@photo) %> |
<%= link_to 'Back', photos_path %>
{% endhighlight %}

<b>app/views/photos/new.html.erb</b>

{% highlight rhtml %}
<h1>New Photo</h1>
<%= render 'form' %>
<%= link_to 'Back', photos_path %>
{% endhighlight %}

<b>app/views/photos/edit.html.erb</b>

{% highlight rhtml %}
<h1>Editing Photo</h1>
<%= render 'form' %>
<%= link_to 'Show', @photo %> |
<%= link_to 'Back', photos_path %>
{% endhighlight %}

<p>The photos app is now finished. Now execute the code below.</p>

{% highlight shell %}
rails s
{% endhighlight %}

<p>Now go to <a href="http://localhost:3000/photos">http://localhost:3000/photos</a>
 and see the rails photo app work. Happy coding!</p>
