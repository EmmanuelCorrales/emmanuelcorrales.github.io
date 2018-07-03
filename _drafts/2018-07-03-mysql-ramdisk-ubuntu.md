---
layout: post
title: "Speed up MySQL database operations on Ubuntu by using the RAM Disk"
date: 2018-07-03 16:30:33 +0800
categories:  mysql ramdisk ubuntu
tags: [ mysql, ramdisk, ubuntu ]
---
I've been working on a project that requires a MySQL database. Everytime I pull
changes from the repository, I have to run the migrations and seed the database.
The seeders contain thousands of rows of data and it takes almost an hour to
finish. A colleague of mine noticed this and taught me a nice trick to speed up
the database operations and that is to use the RAM disk which I'll be sharing in
this post.

Create a directory for the RAM disk.

{% highlight bash %}
sudo mkdir /tmp/ramdisk
{% endhighlight %}

Mount it.

{% highlight bash %}
sudo mount -t tmpfs -o size=2G tmpfs /tmp/ramdisk/
{% endhighlight %}

Move the MySQL databases to the RAM disk.

{% highlight bash %}
sudo mv /var/lib/mysql /tmp/ramdisk/mysql
{% endhighlight %}

Create a symlink to the RAM disk.
{% highlight bash %}
sudo ln -s /tmp/ramdisk/mysql /var/lib/mysql
{% endhighlight %}

Change the permission to allow MySQL to access it.

{% highlight bash %}
sudo chown mysql:mysql /tmp/ramdisk/mysql
{% endhighlight %}

Restart MySQL to apply the changes.

{% highlight bash %}
sudo /etc/init.d/mysql restart
{% endhighlight %}

After moving the databases to the RAM disk running the migrations and seeders
took only a minute to finish.
minute.
