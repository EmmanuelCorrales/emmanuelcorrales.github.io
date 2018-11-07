---
layout: post
title:  "Rails: Deploying to Ubuntu on an EC2 instance with Capistrano, Puma and Nginx."
date:   2018-09-26 08:30:00 +0800
categories: rails ruby ubuntu capistrano puma nginx
tags: [ rails, ruby, ubuntu, capistrano, puma, nginx ]
---

Deploying a Rails application to an EC2 instance with Capistrano.

### Table of Contents
  - [Install Capistrano.](#install_capistrano)
  - [Configure Nginx.](#configure_nginx)
  - [Run initial deployment.](#initial_deployment)

## Setup Ubuntu

{% highlight bash %}
#!/bin/bash

# Script for bootstrapping Ubuntu on EC2.

sudo apt-get update

# Install Nginx.
sudo apt-get install curl git-core nginx -y

# Install Ruby via RVM.
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
curl -sSL https://get.rvm.io | bash -s stable

source /etc/profile.d/rvm.sh
rvm requirements

# Install Ruby.
rvm install 2.2.1
rvm use 2.2.1 --default

# Install Rails and Bundler.
gem install rails -V --no-ri --no-rdoc
gem install bundler -V --no-ri --no-rdoc
{% endhighlight %}

## <a name="setup_rails" />Setup Rails

### Install and configure Capistrano

Add these gems to your Gemfile.

{% highlight ruby %}
group :development do
  # Capistrano for deployment.
  gem 'capistrano',         require: false
  gem 'capistrano-rvm',     require: false
  gem 'capistrano-rails',   require: false
  gem 'capistrano-rails-console',   require: false
  gem 'capistrano-bundler', require: false
  gem 'capistrano3-puma',   require: false
end
{% endhighlight %}

Then install it.

{% highlight bash %}
bundle install
{% endhighlight %}

From your apps local root directory generate the necessary configuration files.
{% highlight bash %}
bundle exec cap install
{% endhighlight %}

This command will generate three files **Capfile**, **config/deploy.rb** and
**config/production.rb**.

Edit the Capfile to look like this:
{% highlight conf %}
# Capfile
# Load DSL and set up stages
require "capistrano/setup"

# Include default deployment tasks
require "capistrano/deploy"

# Load the git SCM plugin appropriate to your project:
require "capistrano/scm/git"
install_plugin Capistrano::SCM::Git

require "capistrano/rvm"
require "capistrano/bundler"
require "capistrano/rails"
require "capistrano/puma"
install_plugin Capistrano::Puma

# Load custom tasks from `lib/capistrano/tasks` if you have any defined
Dir.glob("lib/capistrano/tasks/*.rake").each { |r| import r }
{% endhighlight %}


If I would deploy a Rails app called **example.emmanuelcorrales.com** where the
source code is hosted at **git@github.com:EmmanuelCorrales/example.git** then my
**deploy.rb** at the config directory should look like this:

{% highlight ruby %}
# config/deploy.rb
lock "3.8.1"

set :application,       'example.emmanuelcorrales.com'
set :repo_url,          'git@github.com:EmmanuelCorrales/example.git'
set :user,              'ubuntu'
set :puma_threads,      [4, 16]
set :puma_workers,      0

# Don't change these unless you know what you're doing
set :pty,             true
set :use_sudo,        false
set :stage,           :production
set :deploy_via,      :remote_cache
set :deploy_to,       "/home/#{fetch(:user)}/apps/#{fetch(:application)}"
set :puma_bind,       "unix://#{shared_path}/tmp/sockets/#{fetch(:application)}-puma.sock"
set :puma_state,      "#{shared_path}/tmp/pids/puma.state"
set :puma_pid,        "#{shared_path}/tmp/pids/puma.pid"
set :puma_access_log, "#{release_path}/log/puma.error.log"
set :puma_error_log,  "#{release_path}/log/puma.access.log"
set :ssh_options,     { forward_agent: true, user: fetch(:user), keys: %w(~/.ssh/id_rsa.pub) }
set :puma_preload_app, true
set :puma_worker_timeout, nil
set :puma_init_active_record, true  # Change to false when not using ActiveRecord

## Defaults:
# set :scm,           :git
# set :branch,        :master
# set :format,        :pretty
# set :log_level,     :debug
# set :keep_releases, 5

## Linked Files & Directories (Default None):
set :linked_files, %w{config/application.yml}
set :linked_dirs,  %w{log tmp/pids tmp/cache tmp/sockets vendor/bundle public/system}

namespace :puma do
  desc 'Create Directories for Puma Pids and Socket'
  task :make_dirs do
    on roles(:app) do
      execute "mkdir #{shared_path}/tmp/sockets -p"
      execute "mkdir #{shared_path}/tmp/pids -p"
    end
  end

  before :start, :make_dirs
end

namespace :deploy do
  desc "Make sure local git is in sync with remote."
  task :check_revision do
    on roles(:app) do
      unless `git rev-parse HEAD` == `git rev-parse origin/master`
        puts "WARNING: HEAD is not the same as origin/master"
        puts "Run `git push` to sync changes."
        exit
      end
    end
  end

  desc 'Initial Deploy'
  task :initial do
    on roles(:app) do
      before 'deploy:restart', 'puma:start'
      invoke 'deploy'
    end
  end

  before :starting,     :check_revision
  after  :finishing,    :compile_assets
  after  :finishing,    :cleanup
end

# ps aux | grep puma    # Get puma pid
# kill -s SIGUSR2 pid   # Restart puma
# kill -s SIGTERM pid   # Stop puma
{% endhighlight %}

This is how the *config/deploy/production.rb* would look like.

{% highlight conf %}
# config/deploy/production.rb
server "13.228.29.39",
  user: "ubuntu",
  roles: %w{web app db}
{% endhighlight %}

## Configure Nginx

{% highlight conf %}
upstream puma {
  # Path to Puma SOCK file, as defined previously
  server unix://home/deploy/apps/sponsor.eventsdito.com/shared/tmp/sockets/sponsor.eventsdito.com-puma.sock;
}

server {
  listen 80;
  server_name sponsor.eventsdito.com;

  root /home/deploy/apps/sponsor.eventsdito.com/current/public;
  access_log /home/deploy/apps/sponsor.eventsdito.com/shared/log/nginx.access.log;
  error_log /home/deploy/apps/sponsor.eventsdito.com/shared/log/nginx.error.log info;

  location ~ ^/(assets|fonts|system)/|favicon.ico|robots.txt {
    gzip_static on;
    expires max;
    add_header Cache-Control public;
  }

  try_files $uri/index.html $uri @puma;
  location @puma {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;

    proxy_pass http://puma;
  }

  error_page 500 502 503 504 /500.html;
  client_max_body_size 10M;
  keepalive_timeout 10;
}

{% endhighlight %}

