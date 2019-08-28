---
layout: post
title: "How I redirected all the web traffic for my old domain to the new one"
date: 2019-01-15 08:30:00 +0800
categories: ubuntu, linux, apache, ssl
tags: [ ubuntu, linux, apache, ssl ]
---
I recently decided to change the domain name of my personal website from
*emmanuelcorrales.com* to *emcorrales.com* because it is shorter and easier to
remember. I don't want visitors of *emmanuelcorrales.com* to receive a
*404 Not Found* or any other error so I redirected all the traffic to
*emcorrales.com*. In this article I'll be sharing the steps I took to
accomplish this.

### Steps to redirect web traffic from old domain to the new one:
1. [Launch a server for my old domain.](#launch_a_server_for_the_old_domain)
2. [Setup virtual hosts for the old domain with Apache.](#setup_virtual_hosts_with_apache)
3. [Configure the virtual host to redirect to the new domain.](#configure_virtual_host_to_redirect_to_the_new_domain)
4. [Enable HTTPS for Apache](#enable_https_for_apache)
5. [Install Let's Encrypt SSL Self Signed Certificate for Apache](#install_lets_encrypt)

## <a name="launch_a_server_for_the_old_domain" />Launch a server for my old domain
To assign a server to my old domain, I launched a new instance of an Ubuntu
Server then configured the DNS A records of *emmanuelcorrales.com* which was
hosted at GoDaddy to point to the server's external IP address. This article
will not cover how I configured my DNS records to point to my server since every
registrar provides a unique way of changing their DNS records. You must already
have knowledge on how to configure the DNS with your registrar.

## <a name="setup_virtual_hosts_with_apache" />Setup virtual hosts for the old domain with Apache
Next I created a virtual host for *emmanuelcorrales.com* using the same steps
from a [guide](../blog/apache-setup-a-virtual-host-on-ubuntu) I recently wrote.

## <a name="configure_virtual_host_to_redirect_to_the_new_domain" />Configure the virtual host to redirect to the new domain.
I configured the virtual host for *emmanuelcorrales.com* to redirect to
*https://emcorrales.com* by editing
*/etc/apache2/sites-available/emmanuelcorrales.com.conf*
```bash
sudo vim /etc/apache2/sites-available/emmanuelcorrales.conf
```
to look like this
```conf
<VirtualHost *:80>
    ServerAdmin admin@emmanuelcorrales.com
    ServerName emmanuelcorrales.com
    ServerAlias emmanuelcorrales.com
    DocumentRoot /var/www/emmanuelcorrales.com
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    RedirectMatch permanent ^/(.*)$ https://emcorrales.com/$1
</VirtualHost>
```
The important part to note  here is the line with the *RedirectMatch*. This line
is the configuration I need to redirect traffic from  *emmanuelcorrales.com*
to *https://emcorrales.com*.

I restarted the Apache web server to apply the changes I've made to the
configuration.
```bash
sudo service apache2 restart
```
After restarting, visiting *http://emmanuelcorrales.com* will redirect me to
*https://emcorrales.com*.

## <a name="enable_https_for_apache" />Enable HTTPS for Apache
At this point HTTPS request is still not supported and visiting
*https://emmanuelcorrales.com* causes an error on my browser. To support HTTPS
request I configured Apache to enable SSL for my old domain by running the
following commands:
```bash
sudo a2enmod ssl
sudo a2ensite default-ssl.conf
sudo service apache2 restart
```
After that I was able to access *https://emmanuelcorrales.com* through my
browser. However, the browser warns me that my connection is not private because
I have an invalid certificate. To remove this, I needed to install a certificate
issued by a trusted Certificate Authority but unfortunately I don't want to buy
one so I installed the free certificate provided by *Let's Encrypt*.

## <a name="install_lets_encrypt" />Install Let's Encrypt SSL Self Signed Certificate for Apache
I cloned the repo of *Let's Encrypt* to a directory on my server using Git.
```bash
sudo apt-get -y install git
cd /usr/local
sudo git clone https://github.com/letsencrypt/letsencrypt
```
Then I generated an SSL certifacte for my old domain by running the following
commands below and entering all the required credentials.
```bash
cd /usr/local/letsencrypt
sudo ./letsencrypt-auto --apache -d emmanuelcorrales.com
```
After this the certificate was installed at */etc/letsencrypt/live* and I
confirmed the status of my SSL certificate by visiting the link below.

https://www.ssllabs.com/ssltest/analyze.html?d=emmanuelcorrales.com&latest

And that's it. I have successfully redirected all urls from my old domain to the
new one. If you know a better way to do this please don't hesitate to approach
me.
