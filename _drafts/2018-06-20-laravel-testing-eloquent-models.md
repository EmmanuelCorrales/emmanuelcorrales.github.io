---
layout: post
title:  "Laravel 5.6: Testing Eloquent models"
date:   2018-06-20 16:30:33 +0800
categories: ruby rails
tags: [ ruby, rails ]
---
This past few weeks I've been using Laravel for my recent project. I've been
learning how to apply *Test Driven Development* for Laravel and the first thing
I learned is how to test *Eloquent* models. In this post I'll show how to test
Eloquent models using Laravel 5.6.

Create a new Laravel project.

{% highlight bash %}
composer create-project --prefer-dist laravel/laravel blog
{% endhighlight %}

Make an Article model with a migration.

{% highlight bash %}
php artisan make:model Article -m
{% endhighlight %}

The generated file for Article model will be at *app/Article.php*.

{% highlight php %}
<?php
namespace App;

use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    // No implementation
}
{% endhighlight %}

The generated migration file will be at
*database/migrations/2018_06_18_065913_create_articles_table.php*.
An article table should have the columns title, content and views. The migration
should look like this:

{% highlight php %}
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateArticlesTable extends Migration
{
    public function up()
    {
        Schema::create('articles', function (Blueprint $table) {
            $table->increments('id');
            $table->timestamps();

            $table->string('title');
            $table->text('content');
            $table->integer('views')->default(0);
        });
    }

    public function down()
    {
        Schema::dropIfExists('articles');
    }
}
{% endhighlight %}

Run the migration for creating a table for Articles.

{% highlight bash %}
php artisan migrate
{% endhighlight %}


Make a factory for the Article model.

{% highlight bash %}
php artisan make:factory ArticleFactory --model=Article
{% endhighlight %}

The generated factory for the model Article will be at
*database/factories/ArticleFactory.php*.

{% highlight php %}
<?php

use Faker\Generator as Faker;

$factory->define(App\Article::class, function (Faker $faker) {
    return [
        'title' => $faker->name,
        'content' => $faker->sentence
    ];
});
{% endhighlight %}

Create a directory for integration tests at
*tests/Integration/models/ArticleTest.php*.

{% highlight bash %}
mkdir -p tests/Integration/models
touch tests/Integration/models/ArticleTest.php
{% endhighlight %}

Run phpunit.
{% highlight bash %}
./vendor/bin/phpunit tests/Integration/ArticleTest
{% endhighlight %}
