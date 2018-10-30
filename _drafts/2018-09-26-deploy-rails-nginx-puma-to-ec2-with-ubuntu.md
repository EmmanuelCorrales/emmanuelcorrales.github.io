---
layout: post
title:  "Rails: Deploying to Ubuntu on an EC2 instance with Capistrano, Puma and Nginx."
date:   2018-09-26 08:30:00 +0800
categories: rails ruby ubuntu capistrano puma nginx
tags: [ rails, ruby, ubuntu, capistrano, puma, nginx ]
---

Deploying a Rails application to an EC2 instance with Capistrano.

### Table of Contents
- [Setup EC2](#setup_ec2)
  - [Setup VPC(Virtual Private Cloud)](#setup_vpc)
    - [Create a subnet.](#create_subnet)
    - [Allow internet access via Internet Gateway.](#allow_internet)
    - [Make the subnet public.](#public_subnet)
    - [Enable public IP and DNS host names assignment on EC2 launch.](#enable_public_ip_dns_hostname)
    - [Allow HTTP and SSH traffic through security groups.](#allow_http_ssh)
  - [Find the latest image of Ubuntu 16.04 LTS.](#find_latest_ubuntu)
  - [Create a key pair.](#create_key_pair)
  - [Launch EC2.](#launch_ec2)
  - [Accessing Ubuntu on EC2](#access_ec2)
  - [Automatically configure the Ubuntu Server on launch](#automatic_server_configuration)
  - [Launch a bootstrapped EC2 instance.](#launch_bootstrapped_ec2)
- [Prepare Rails app for deployment.](#setup_rails)
  - [Install Capistrano.](#install_capistrano)
  - [Configure Nginx.](#configure_nginx)
  - [Run initial deployment.](#initial_deployment)

## <a name="setup_ec2" />Setup EC2
Before we can deploy our Rails app we must first set up the AWS Services we are
going to use. We will be deploying our Rails app to an EC2 instance of Ubuntu
so well be launching one before deploying our Rails app. There are lots of
prerequites to launch an EC2 instance like a subnet, a security group, and an
image id. We will also automatically configure our server for deployment upon
launch. The server configuration will involve using services like S3 to store
our ssh keys for our git remote repository as well as setting up the Ruby on
Rails environment.

### <a name="setup_vpc" />Setup VPC(Virtual Private Cloud)
We are going to create a new VPC and use it for our Rails app instead of the
default one provided by AWS. A VPC is like your home network but on the cloud.
A VPC can have multiple subnets which is like a router and a hotspot at home. A
subnet can have multiple EC2 instances running. Lets setup the VPC.

Execute the command below to create a VPC. If successful the command will
respond with the id for the newly created VPC.
{% highlight bash  %}
aws ec2 create-vpc --cidr-block 10.0.0.0/28 | jq -r '.Vpc.VpcId'
# vpc-046ea804c737b0734
{% endhighlight %}

#### <a name="create_subnet" />Create a subnet.

An EC2 instance needs a subnet to be launched. Create a subnet by executing the
command below.
{% highlight bash %}
aws ec2 create-subnet --vpc-id vpc-046ea804c737b0734 --cidr-block 10.0.0.0/28 \
  | jq -r '.Subnet.SubnetId'
# subnet-0b9e54cc7752c940e
{% endhighlight %}

We have created a subnet inside the VPC but we still can't launch an EC2
instance. Well we can but we won't, not yet, because the subnet we created is
**private** and not accessible from outside the AWS infrastructure. We have to
make this subnet **public**. We must first make the VPC accessible from outside
AWS infrastructure.

#### <a name="allow_internet" />Allow internet access via Internet Gateway
We are going to create and attach an Internet Gateway to our VPC. Execute the
command below to create an internet gateway. It should return the id
of the internet gateway if it was successfully created.

{% highlight bash %}
aws ec2 create-internet-gateway | jq -r '.InternetGateway.InternetGatewayId'
# igw-0ca0f555df971303c
{% endhighlight %}

Attach the internet gateway to the VPC we created earlier by providing both the
vpc id and internet gateway id to the command below.
{% highlight bash %}
aws ec2 attach-internet-gateway --vpc-id vpc-046ea804c737b0734 \
  --internet-gateway-id igw-0ca0f555df971303c --region ap-southeast-1
{% endhighlight %}

Now that our VPC is accessible outside the AWS Infrastructure, we are going to
route the subnet properly to make it public.
#### <a name="public_subnet" />Make the subnet public.

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

Now the internet traffic can pass through the VPC and can be routed to the
subnet but if we launched an EC2 instance how can we access it? We need to know
the EC2 instance's public DNS but where can we find it? We can find the public
DNS hostname on an EC2 instance's attribute but unfortunately the VPC and the
subnet is not yet configured to assign a public DNS hostname to an EC2 instance.

#### <a name="enable_public_ip_dns_hostname" />Enable public IP and DNS host names assignment on EC2 launch

By default a subnet is configured not to auto-assign a public ip after
launching a new EC2 instance. Modify the subnet attribute to auto-assign a
public ip whenever an EC2 instance is launched.

{% highlight bash %}
aws ec2 modify-subnet-attribute --map-public-ip-on-launch \
  --subnet-id subnet-0b9e54cc7752c940e
{% endhighlight %}

EC2 instances launched on non-default VPC don't get hostnames by default.
We need to modify the vpc attribute to allow automatic DNS hostnames assignment.
{% highlight bash %}
aws ec2 modify-vpc-attribute --vpc-id vpc-a01106c2 \
  --enable-dns-hostnames "{\"Value\":true}"
{% endhighlight %}

Now every EC2 instance launched on the subnet subnet-0b9e54cc7752c940e will
have a public DNS hostname assigned to them. Unfortunately we still won't launch
an EC2 instance because we won't be able to access it using the HTTP and SSH
protocol.

#### <a name="allow_http_ssh" />Allow HTTP and SSH access via Security Groups
We will allow access to the EC2 instance through SSH and HTTP protocols by
creating a security group and whitelisting those protocols. A security group is
also required to launch an EC2 instance.

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

We have now finished setting up the VPC for our Rails app.

### <a name="find_latest_ubuntu" />Find the latest image of Ubuntu

Launching an EC2 instance of Ubuntu through the terminal requires an image id as
argument. We can find the id of the latest AMI(Amazon Machine Image) for Ubuntu
16.04 LTS by executing the command below.

{% highlight bash %}
aws ec2 describe-images --owners 099720109477 \
  --filters 'Name=state,Values=available' \
  'Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-xenial-16.04-amd64-server-????????' \
  --output json | jq -r '.Images | sort_by(.CreationDate) | last(.[]).ImageId'
{% endhighlight %}

The **owners** parameter is the id of the vendor called Canonical which is the
company behind Ubuntu. If successful the command would return the latest id of
the AMI for Ubuntu. At the time of writing, the id latest image for Ubuntu
16.04 LTS Server is **ami-0d97809b54a5f01ba**.

### <a name="create_key_pair" />Create a key pair.

An EC2 instance requires a key pair. The key pair is used to access running EC2
instances via SSH. Create a key pair by executing the command below.

{% highlight bash %}
aws ec2 create-key-pair --key-name RailsEC2 | jq -r '.KeyMaterial' > RailsEC2.pem
{% endhighlight %}

### <a name="launch_ec2" />Launching the EC2 instance.
Now that we have all we need to launch an EC2 instance of Ubuntu lets execute
the command below to launch one new EC2 instance.

{% highlight bash %}
aws ec2 run-instances --count 1 --instance-type t2.micro \
  --key-name RailsEC2 \
  --image-id ami-0d97809b54a5f01ba \
  --security-group-ids sg-01b68dc626cc61562 \
  --subnet-id subnet-0b9e54cc7752c940e \
  | jq -r '.Instances[].InstanceId'
# i-0eb7fac3a8e066e77"
{% endhighlight %}

If successful, the command above would return the id of the new instance.
Lets now check if we can access the EC2 instance we launched.

### <a name="access_ec2" />Accessing Ubuntu on EC2.

First get the the public DNS hostname of the instance we launched by running the
command below. You may have to wait a few minutes after launching the EC2
instance for this command to work because the command would only work on
instances with **running** status and not with **pending**.

{% highlight bash %}
aws ec2 describe-instances --instance-ids i-0eb7fac3a8e066e77 \
  | jq -r '.Reservations[].Instances[].PublicDnsName'
{% endhighlight %}

Run the command below and pass the location of the pem file we created earlier:

{% highlight bash %}
ssh -i "RailsEC2.pem" ubuntu@ec2-13-251-103-23.ap-southeast-1.compute.amazonaws.com
{% endhighlight %}

If the command is successful then you'll be logged in to your Ubuntu Server and
we can now manually setup the Ruby on Rails environment for deployment. The
summary of steps for configuring the server are as follows:

1. Get the latest updates for Ubuntu.
2. Install Curl, Git and Nginx.
3. Download and install RVM.
4. Download and install Ruby via RVM.
5. Install Rails.
6. Install Bundler.

That looks simple and easy to do but what if we are launching twenty instances?
Should we configure it twenty times as well? Of course not! That is extremely
tedious! We are going to automate it.

### <a name="automatic_server_configuration" />Automatically configure the Ubuntu Server on launch.

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

source /etc/profile.d/rvm.sh
rvm requirements

# Install Ruby.
rvm install 2.2.1
rvm use 2.2.1 --default

# Install Rails and Bundler.
gem install rails -V --no-ri --no-rdoc
gem install bundler -V --no-ri --no-rdoc
{% endhighlight %}

The comments on the script explains what each line does.
### <a name="launch_bootstrapped_ec2" />Launching the bootstrapped EC2 instance.
Launch a new EC2 instance with an additional argument **user-data** where the
value is the location of the bootstrap script.

{% highlight bash %}
aws ec2 run-instances --count 1 --instance-type t2.micro \
  --key-name RailsEC2 \
  --image-id ami-0d97809b54a5f01ba \
  --security-group-ids sg-01b68dc626cc61562 \
  --subnet-id subnet-0b9e54cc7752c940e \
  --user-data file://bootstrap.sh
{% endhighlight %}

Login to the server via ssh.

### <a name="access_ec2" />Confirm Rails installation
List all the instances public DNS.

{% highlight bash %}
aws ec2 describe-instances | jq -r '.Reservations[].Instances[].PublicDnsName'
# ubuntu@ec2-13-251-103-24.ap-southeast-1.compute.amazonaws.com
{% endhighlight %}

Choose the DNS hostname of the newly launched instance then pass it and the
location of the same pem file we used earlier as arguments to the ssh command
like the example below:
{% highlight bash %}
ssh -i "RailsEC2.pem" ubuntu@ec2-13-251-103-24.ap-southeast-1.compute.amazonaws.com
{% endhighlight %}

Check if Rails was installed sucessfully.
{% highlight bash %}
rails -v
{% endhighlight %}

If there is an error you can investigate what went wrong by reading the logs at
*/var/log/cloud-init-output.log*.
{% highlight bash %}
less /var/log/cloud-init-output.log
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

