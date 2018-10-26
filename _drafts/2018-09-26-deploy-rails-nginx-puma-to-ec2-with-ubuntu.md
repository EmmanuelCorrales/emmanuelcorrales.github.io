---
layout: post
title:  "Rails: Deploying to Ubuntu on an EC2 instance with Capistrano, Puma and Nginx."
date:   2018-09-26 08:30:00 +0800
categories: rails ruby ubuntu capistrano puma nginx
tags: [ rails, ruby, ubuntu, capistrano, puma, nginx ]
---

Deploying a Rails application to an EC2 instance with Capistrano.

### Table of Contents
- [Setup infrastructure.](#setup_infrastructure)
- [Activate MFA on the root account.](#activate_mfa)
- [Prefer an admin user over a root user.](#replace_root_user)
- [Setup the AWS CLI](#setup_aws_cli)


## <a name="setup_infrastructure" />Setup the infrastructure

### Setup S3 bucket
Create an S3 bucket to store the private keys. Make sure that the bucket name is
globally unique.
{% highlight bash %}
aws s3api create-bucket --bucket my-bucket --region ap-southeast-1 \
  --create-bucket-configuration LocationConstraint=ap-southeast-1 \
  | jq -r '.Location'
# http://my-bucket.s3.amazonaws.com/
{% endhighlight %}


Check if your bucket was succesfully created.
{% highlight bash %}
aws s3 ls
# 2018-10-26 17:36:38 my-bucket
{% endhighlight %}

The command above will include the creation date and name of your bucket to its
output if it was created successfully.

### Setup SSH with Github.
The Ubuntu server we will launch on EC2 needs to have access on the Rails app's
repository to be able to fetch updates. We are going to set up SSH credentials.
We are going to assume that the repository is Github.

Create the private and public key by running the command below.
{% highlight bash %}
ssh-keygen -t rsa -f github_key
{% endhighlight %}

This will generate two files, **github_key** and **github_key.pub**.
We add **github_key.pub** as the SSH key to our Github account. Please refer
[here](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/).

Upload it to the bucket we created earlier

{% highlight bash %}
aws s3 cp github_key s3://my-bucket/
# upload: ./github_key to s3://my-bucket/github_key
{% endhighlight %}

The EC2 instance of the Ubuntu server can download it upon launch with the
execution of the **bootstap script** which we will cover on the next section.

### Bootstrap script
A bootstrap script is a script that is executed after the initial launch of an EC2
instance. We are going to create and use the bootstrap script to automatically
configure the production environment for our Rails app everytime a new EC2
instance of Ubuntu Server is launched.

Create a bootstrap script called **bootstrap.sh**.

{% highlight bash %}
#!/bin/bash

# Script for bootstrapping Ubuntu on EC2.

sudo apt-get update

# Install Nginx.
sudo apt-get install curl git-core nginx -y

# Install Ruby via RVM.
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
curl -sSL https://get.rvm.io | bash -s stable

source ~/.rvm/scripts/rvm
rvm requirements

# Install Ruby.
rvm install 2.2.1
rvm use 2.2.1 --default

# Install Rails and Bundler.
gem install rails -V --no-ri --no-rdoc
gem install bundler -V --no-ri --no-rdoc

# Download the ssh key to github
curl https://s3.amazonaws.com/my-bucket/github_key

# Add the ssh key to the ssh-agent.
ssh-add github_key
{% endhighlight %}


### Launch an EC2 instance of the Ubuntu Server

Create a VPC.
{% highlight bash  %}
aws ec2 create-vpc --cidr-block 10.0.0.0/28 | jq -r '.Vpc.VpcId'
# vpc-046ea804c737b0734
{% endhighlight %}

If successful the command will output the id for the created VPC.

Create a subnet.

{% highlight bash %}
aws ec2 create-subnet --vpc-id vpc-046ea804c737b0734 --cidr-block 10.0.0.0/28 \
  | jq -r '.Subnet.SubnetId'
# subnet-0b9e54cc7752c940e
{% endhighlight %}

Create an internet gateway.
{% highlight bash %}
aws ec2 create-internet-gateway | jq -r '.InternetGateway.InternetGatewayId'
# igw-0ca0f555df971303c
{% endhighlight %}

then attach it to the VPC we created earlier.
{% highlight bash %}
aws ec2 attach-internet-gateway --vpc-id vpc-046ea804c737b0734 \
  --internet-gateway-id igw-0ca0f555df971303c --region ap-southeast-1
{% endhighlight %}

Create a route table.

{% highlight bash %}
aws ec2 create-route-table --vpc-id vpc-046ea804c737b0734 \
  | jq -r '.RouteTable.RouteTableId'
# rtb-0da48374f8b6cec1d
{% endhighlight %}

Create a route to the internet gateway.
{% highlight bash %}
aws ec2 create-route --route-table-id rtb-0da48374f8b6cec1d \
  --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0ca0f555df971303c
{% endhighlight %}

Associate the route table to the subnet.
{% highlight bash %}
aws ec2 associate-route-table --route-table-id rtb-0da48374f8b6cec1d \
  --subnet-id subnet-0b9e54cc7752c940e | jq -r '.AssociationId'
# rtbassoc-0e055803e90e416c7
{% endhighlight %}

Create a security group.
{% highlight bash %}
aws ec2 create-security-group \
  --group-name rails-deploy-ec2 \
  --description "Rails deployment to an EC2 instance." \
  --vpc-id vpc-046ea804c737b0734 | jq -r '.GroupId'
# sg-01b68dc626cc61562
{% endhighlight %}

Allow access via HTTP on port 80 and SSH on port 22.
{% highlight bash %}
aws ec2 authorize-security-group-ingress --group-id sg-01b68dc626cc61562 \
    --ip-permissions \
    IpProtocol=tcp,FromPort=22,ToPort=22,IpRanges=[{CidrIp=0.0.0.0/0}] \
    IpProtocol=tcp,FromPort=80,ToPort=80,IpRanges=[{CidrIp=0.0.0.0/0}]
{% endhighlight %}

Create a key pair. This key pair is used to access running EC2 instances via
SSH.
{% highlight bash %}
aws ec2 create-key-pair --key-name RailsEC2 | jq -r '.KeyMaterial' > RailsEC2.pem
{% endhighlight %}


Find the latest Ubuntu image.

{% highlight bash %}
aws ec2 describe-images --owners 099720109477 \
  --filters 'Name=state,Values=available' \
  'Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-xenial-16.04-amd64-server-????????' \
  --output json | jq -r '.Images | sort_by(.CreationDate) | last(.[]).ImageId'
# ami-0d97809b54a5f01ba
{% endhighlight %}

Launch an EC2 instance.

{% highlight bash %}
aws ec2 run-instances --count 1 --instance-type t2.micro \
  --key-name RailsEC2 \
  --image-id ami-0d97809b54a5f01ba \
  --security-group-ids sg-01b68dc626cc61562 \
  --user-data file://bootstrap.sh \
  --subnet-id subnet-0b9e54cc7752c940e
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

