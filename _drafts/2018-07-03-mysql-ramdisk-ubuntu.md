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

## Why use the RAM?
RAM(Random Access Memory) is much faster to read from and write to than the
other kinds of storage in a computer, such as the hard disk/ssd where MySQL
stores the database by default. Moving the database to the RAM can dramatically
increase the input and output speed of database operations.

## Drawbacks!
RAM is a volatile memory which means it requires power to retain the data it
stores. This means that the data stored on the RAM will perish once the computer
is turned off. In my case, I use the database only for testing and development
on my local machine. If I ever needed to retain the data for testing, I'll just
backup the database then restore it the next time I need it.

## Moving the database to the RAM.
Now that you know the advantages and disadvantages of moving the MySQL databases
on the RAM disk, I'll be walking you through the steps on how to achieve this.

First we backup all our databases.

{% highlight bash %}
mysqldump -u root -p --all-databases > alldb_backup.sql
{% endhighlight %}

Create a directory for the RAM disk.

{% highlight bash %}
sudo mkdir /tmp/ramdisk
{% endhighlight %}

Mount it. I assigned a size of 2GB for the ramdisk. Its up to you how much space
you want, just make sure it can accomodate all the data you will write to the
database.

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

Now we're finished! After moving the databases to the RAM disk running the
migrations and seeders took only a minute to finish compared to almost an hour
when using the hard disk.


## Restoring the database.

Since the database is saved in the RAM disk the database will be gone everytime
the device is turned off.

Delete symlink to mysql ramdisk.
{% highlight bash %}
sudo rm -rf /var/lib/mysql
{% endhighlight %}

Copy the database from the backup.
{% highlight bash %}
sudo cp -pRL /var/lib/mysql_backup/mysql /var/lib/
{% endhighlight %}
