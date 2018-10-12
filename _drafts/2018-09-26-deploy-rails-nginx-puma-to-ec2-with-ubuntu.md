---
layout: post
title:  "Rails: Deploying to Ubuntu on an EC2 instance with Capistrano, Puma and Nginx."
date:   2018-09-26 08:30:00 +0800
categories: rails ruby ubuntu capistrano puma nginx
tags: [ rails, ruby, ubuntu, capistrano, puma, nginx ]
---

Deploying a Rails application to an EC2 instance with Capistrano.


## Create an EC2 instance.
Find the latest AMI(Amazon Machine Image) of ubuntu 16.04.

{% highlight bash %}
aws ec2 describe-images --owners 099720109477 \
  --filters 'Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-xenial-16.04-amd64-server-????????' \
            'Name=state,Values=available' \
  --output json \
| jq -r '.Images | sort_by(.CreationDate) | last(.[]).ImageId'
{% endhighlight %}

Create a VPC.

{% highlight bash  %}
aws ec2 create-vpc --cidr-block 10.0.0.0/16
{% endhighlight %}

It should return this:
{% highlight json  %}
{
  "Vpc": {
      "VpcId": "vpc-ff7bbf86",
      "InstanceTenancy": "default",
      "Tags": [],
      "CidrBlockAssociations": [
          {
              "AssociationId": "vpc-cidr-assoc-6e42b505",
              "CidrBlock": "10.0.0.0/16",
              "CidrBlockState": {
                  "State": "associated"
              }
          }
      ],
      "Ipv6CidrBlockAssociationSet": [],
      "State": "pending",
      "DhcpOptionsId": "dopt-38f7a057",
      "CidrBlock": "10.0.0.0/16",
      "IsDefault": false
  }
}
{% endhighlight %}

Create a security group for our new VPC.

{% highlight bash %}
aws ec2 create-security-group --group-name rails-deploy-ec2 \
  --description "Security group for deploying a Rails application on an EC2 instance." \
  --vpc-id vpc-ff7bbf86
{% endhighlight %}

## Setup Capistrano

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

Generate the necessary configuration files.

{% highlight bash %}
bundle exec cap install
{% endhighlight %}

This will generate a Capfile.

{% highlight conf %}
# Capfile
# Load DSL and set up stages
require "capistrano/setup"

# Include default deployment tasks
require "capistrano/deploy"

# Load the SCM plugin appropriate to your project:
#
# require "capistrano/scm/hg"
# install_plugin Capistrano::SCM::Hg
# or
# require "capistrano/scm/svn"
# install_plugin Capistrano::SCM::Svn
# or
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

This will generate a **deploy.rb** file at the *config* directory.

{% highlight ruby %}
# config/deploy.rb
lock "3.8.1"

set :application,       'sponsor.eventsdito.com'
set :repo_url,          'git@bitbucket.org:krumbsph/sponsor.eventsdito.com.git'
set :user,              'deploy'
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
  user: "deploy",
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

