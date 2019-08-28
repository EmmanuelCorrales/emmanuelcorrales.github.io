---
layout: post
title: "Apache: Setup a virtual host on Ubuntu 18.04 LTS"
date: 2019-01-09 08:30:00 +0800
categories: ubuntu, linux
tags: [ ubuntu, linux, apache ]
---

This guide is about setting up a virtual host on Ubuntu 18.04 LTS with Apache as
the web server. We'll assume that we want to host a simple site where the domain
name will be *example.com*.

Install the Apache Web Server.
```bash
sudo apt-get update
sudo apt-get install apache2
```

Create a directory called *example.com* inside the */var/www* and make sure that
the current user can access it by changing its owner and permission.
```bash
sudo mkdir -p /var/www/example.com
sudo chown -R $USER:$USER /var/www/example.com
sudo chmod -R 755 /var/www
```
Create an index file which will be the entry point of the website. Run the
command below to open an editor.
```bash
vim /var/www/example.com/index.html
```

Edit the contents to look like the code below.
```html
<html>
  <body>
    <h1>The virtual host is working</h1>
  </body>
</html>
```

Make the site available for Apache by copying the default configuration to our
site's own configuration.
```bash
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/example.com.conf
```

Open the copied configuration file with a text editor.
```bash
sudo vim /etc/apache2/sites-available/example.com.conf
```
Edit the contents to look like below.
```conf
<VirtualHost *:80>
    ServerAdmin webmaster@example.com
    DocumentRoot /var/www/example.com
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

We need to disable Apache's default configuration so it won't have any conflict
with our configuration.
```bash
sudo a2dissite 000-default.conf
```
Enable our own config file by running the command below.
```bash
sudo a2ensite example.com.conf
```
Restart the Apache web server to apply the changes.
```bash
sudo systemctl restart apache2
```

Add *example.com* to the virtual hosts by editing */etc/hosts*.
```bash
sudo vim /etc/hosts
```
and add this line
```conf
127.0.0.1 example.com
```
Test by using the curl command.
```
curl --header "hostname: example.com" example.com
```
It should respond with the contents of */var/www/example.com/index.html*.

You may also type *example.com* in the browser and see the message "*The virtual
host is working.*" if you set it up correctly.
