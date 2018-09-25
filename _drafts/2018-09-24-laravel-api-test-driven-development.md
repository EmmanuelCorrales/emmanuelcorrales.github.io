---
layout: post
title:  "Laravel: Building an API with Test Driven Development approach"
date:   2018-09-24 08:30:00 +0800
categories: laravel api tdd php
tags: [ laravel, api, tdd, php ]
---
Applying test driven development to a laravel API.

Let's start creating a new Laravel project with:

{% highlight bash %}
laravel new blog
{% endhighlight %}

or

{% highlight bash %}
composer create-project --prefer-dist laravel/laravel blog
{% endhighlight %}

## Prepare a testing environment.

Lets first setup our testing environment by changing the contents of the
**.env** file with the right credentials. Below is the default configuration of
the database.

{% highlight yaml %}
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
{% endhighlight %}

I don't want to use  *homestead* as the database, I prefer to use *blog*. I also
want to use *blog_test* as the database I'll use for testing.

{% highlight yaml %}
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=blog
DB_DATABASE_TEST=blog_test
DB_USERNAME=homestead
DB_PASSWORD=secret
{% endhighlight %}

This is our original *config/database.php*.

{% highlight php %}
// config/database.php
'connections' => [
    'mysql' => [
       'driver' => 'mysql',
       'host' => env('DB_HOST', '127.0.0.1'),
       'port' => env('DB_PORT', '3306'),
       'database' => env('DB_DATABASE', 'forge'),
       'username' => env('DB_USERNAME', 'forge'),
       'password' => env('DB_PASSWORD', ''),
       'unix_socket' => env('DB_SOCKET', ''),
       'charset' => 'utf8mb4',
       'collation' => 'utf8mb4_unicode_ci',
       'prefix' => '',
       'strict' => true,
       'engine' => null,
    ],
],
{% endhighlight %}

We'll modify it to add configuration for the test database.

{% highlight php %}
// config/database.php
'connections' => [
    'mysql_testing' => [
       'driver' => 'mysql',
       'host' => env('DB_HOST', '127.0.0.1'),
       'port' => env('DB_PORT', '3306'),
       'database' => env('DB_DATABASE_TEST', 'forge'),
       'username' => env('DB_USERNAME', 'forge'),
       'password' => env('DB_PASSWORD', ''),
       'unix_socket' => env('DB_SOCKET', ''),
       'charset' => 'utf8mb4',
       'collation' => 'utf8mb4_unicode_ci',
       'prefix' => '',
       'strict' => true,
       'engine' => null,
    ],
    'mysql' => [
       'driver' => 'mysql',
       'host' => env('DB_HOST', '127.0.0.1'),
       'port' => env('DB_PORT', '3306'),
       'database' => env('DB_DATABASE', 'forge'),
       'username' => env('DB_USERNAME', 'forge'),
       'password' => env('DB_PASSWORD', ''),
       'unix_socket' => env('DB_SOCKET', ''),
       'charset' => 'utf8mb4',
       'collation' => 'utf8mb4_unicode_ci',
       'prefix' => '',
       'strict' => true,
       'engine' => null,
    ],
],
{% endhighlight %}

This is our origial *phpunit.xml*.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<phpunit backupGlobals="false"
         backupStaticAttributes="false"
         bootstrap="vendor/autoload.php"
         colors="true"
         convertErrorsToExceptions="true"
         convertNoticesToExceptions="true"
         convertWarningsToExceptions="true"
         processIsolation="false"
         stopOnFailure="false">
    <testsuites>
        <testsuite name="Unit">
            <directory suffix="Test.php">./tests/Unit</directory>
        </testsuite>

        <testsuite name="Feature">
            <directory suffix="Test.php">./tests/Feature</directory>
        </testsuite>
    </testsuites>
    <filter>
        <whitelist processUncoveredFilesFromWhitelist="true">
            <directory suffix=".php">./app</directory>
        </whitelist>
    </filter>
    <php>
        <env name="APP_ENV" value="testing"/>
        <env name="BCRYPT_ROUNDS" value="4"/>
        <env name="CACHE_DRIVER" value="array"/>
        <env name="SESSION_DRIVER" value="array"/>
        <env name="QUEUE_DRIVER" value="sync"/>
        <env name="MAIL_DRIVER" value="array"/>
    </php>
</phpunit>
{% endhighlight %}


First we'll create an Article model with migration, controller and resource.
{% highlight bash %}
php artisan make:model Article -mcr
{% endhighlight %}

