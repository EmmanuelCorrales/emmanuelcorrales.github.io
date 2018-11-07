---
layout: post
title:  "Rails: Capistrano for easy deployment and configuration to Ubuntu with
Nginx and Puma"
date:   2018-09-26 08:30:00 +0800
categories: rails ruby ubuntu capistrano puma nginx
tags: [ rails, ruby, ubuntu, capistrano, puma, nginx ]
---

In this post I'll demonstrate how to use Capistrano to deploy and configure a
Rails app to an Ubuntu Server with Nginx and Puma.

### Table of Contents
- [Setup Ubuntu.](#setup_ubuntu)
- [Setup Capistrano.](#setup_capistrano)
- [Configure Nginx.](#configure_nginx)

## <a name="setup_ubuntu" />Setup Ubuntu

Ssh to the server.
```bash
ssh emmanuelcorrales.com
```

Create user **deploy**.
```bash
adduser deploy
```

Change its password.
```bash
passwd deploy
```

Update Ubuntu then install Git and Nginx.
```bash
sudo apt-get update
sudo apt-get install curl git-core nginx -y
```

Install RVM.
```bash
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
curl -sSL https://get.rvm.io | bash -s stable
source /etc/profile.d/rvm.sh
rvm requirements
```

Install Ruby via RVM.
```bash
rvm install 2.2.1
rvm use 2.2.1 --default
```

Install Rails and Bundler without docs too save space and nobody needs docs on
the server.
```bash
gem install rails -v '5.2.0' -V --no-ri --no-rdoc
gem install bundler -V --no-ri --no-rdoc
```

## <a name="setup_capistrano" />Setup Capistrano

### "Capify" the Rails app.

Add this gem to your Gemfile.
```ruby
group :development do
  gem 'capistrano', '~>3.10', require: false
end
```
Then install it.
```bash
bundle install
```
From your apps local root directory generate the necessary configuration files.
```bash
bundle exec cap install
```
This command will generate four files **Capfile**, **config/deploy.rb**
**config/deploy/staging.rb** and **config/deploy/production.rb**.

We want to deploy the Rails app called **example.emmanuelcorrales.com** whose
repository is hosted at **git@github.com:EmmanuelCorrales/example.git**.
Different verisons of the app would be stored at the
**/home/default/example.emmanuelcorrales.com** directory. Configure Capistrano
to install it there by changing the contents of  **config/deploy.rb** to look
like the code below.
```ruby
# config valid for current version and patch releases of Capistrano
lock "~> 3.11.0"

set :application, 'example.emmanuelcorrales.com'
set :repo_url, 'git@github.com:EmmanuelCorrales/example.git'

# Deploy configurations
set :deploy_via,      :remote_cache
set :deploy_to,       "/home/#{fetch(:user)}/apps/#{fetch(:application)}"
```
### Add Ruby on Rails deployment tasks

Add this gem to your Gemfile under the capistrano gem.
```ruby
gem 'capistrano-rails', '~>1.4', require: false
```

Then install it.
```bash
bundle install
```

Edit the **Capfile** to look like this:

```conf
# Load DSL and set up stages
require "capistrano/setup"

# Include default deployment tasks
require "capistrano/deploy"

# Load the git SCM plugin appropriate to your project:
require "capistrano/scm/git"
install_plugin Capistrano::SCM::Git

require "capistrano/rails"

# Load custom tasks from `lib/capistrano/tasks` if you have any defined
Dir.glob("lib/capistrano/tasks/*.rake").each { |r| import r }
```

Symlink Rails shared files and directories like log, tmp and public/uploads.
Enable it by setting the **linked_dirs** and **linked_files** options at the
**deploy.rb**.

```ruby
## Linked Files & Directories (Default None):
append :linked_dirs, 'log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'vendor/bundle', '.bundle', 'public/system', 'public/uploads'
append :linked_files, 'config/database.yml', 'config/secrets.yml', 'config/application.yml'
```

### Configure Puma

Add this gem to your Gemfile under the capistrano gem.
```ruby
gem 'capistrano3-puma', require: false
```

Then install it.
```bash
bundle install
```

Add these lines to your Capfile.
```ruby
require 'capistrano/puma'
install_plugin Capistrano::Puma
```
Add these lines to the **config/deploy.rb**.
```ruby
# Puma configurations
set :puma_threads, [4, 16]
set :puma_workers, 0
set :puma_bind, "unix://#{shared_path}/tmp/sockets/#{fetch(:application)}-puma.sock"
set :puma_state, "#{shared_path}/tmp/pids/puma.state"
set :puma_pid, "#{shared_path}/tmp/pids/puma.pid"
set :puma_access_log, "#{release_path}/log/puma.error.log"
set :puma_error_log, "#{release_path}/log/puma.access.log"
set :puma_preload_app, true
set :puma_worker_timeout, nil
set :puma_init_active_record, true
```

### Configure Nginx

Add this line to your Capfile below the line *install_plugin Capistrano::Puma*.
```ruby
install_plugin Capistrano::Puma
```

Add these lines to the **config/deploy.rb**. These are the parameters for the
Nginx configurations that will be uploaded to the server.
```ruby
# Nginx configurations
set :nginx_config_name, "#{fetch(:application)}_#{fetch(:stage)}"
set :nginx_flags, 'fail_timeout=0'
set :nginx_http_flags, fetch(:nginx_flags)
set :nginx_server_name, "localhost #{fetch(:application)}.local"
set :nginx_sites_available_path, '/etc/nginx/sites-available'
set :nginx_sites_enabled_path, '/etc/nginx/sites-enabled'
set :nginx_socket_flags, fetch(:nginx_flags)
set :nginx_ssl_certificate, "/etc/ssl/certs/#{fetch(:nginx_config_name)}.crt"
set :nginx_ssl_certificate_key, "/etc/ssl/private/#{fetch(:nginx_config_name)}.key"
set :nginx_use_ssl, false
```

Upload the nginx configuration to the server.
```bash
bundle exec cap production puma_nginx:config
```
## Deploy the Rails app.

Edit the contents of **config/deploy/production.rb** to look like this:
```ruby
server "example.emmanuelcorrales.com", user: "deploy", roles: %w{web app db}
```

Deploy to the server.

```bash
bundle exec cap production deploy
```
