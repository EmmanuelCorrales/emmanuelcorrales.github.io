---
layout: post
title:  "Laravel 5.6: Testing models"
date:   2018-06-20 16:30:33 +0800
categories: laravel php eloquent tdd
tags: [ laravel, php, eloquent, tdd ]
---
These past few weeks I've been using Laravel at work. I've been studying how to
apply *Test Driven Development* for Laravel and the first thing I learned is how
to test models in which I'll demonstrate using Laravel 5.6 by creating a new
project which has an Article model with attributes *title*, *content* and
*views*.

Let's start by creating a new Laravel project with:

{% highlight bash %}
laravel new blog
{% endhighlight %}

or

{% highlight bash %}
composer create-project --prefer-dist laravel/laravel blog
{% endhighlight %}

## Setup the Article model and the database table.

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
    // No implementation yet
}
{% endhighlight %}


The articles table should have columns *title*, *content* and *views*. The
generated migration file will be at
*database/migrations/2018_06_18_065913_create_articles_table.php*.

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

            table->string('title');
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

Run the migration for creating the table for articles.

{% highlight bash %}
php artisan migrate
{% endhighlight %}

## Creating for the Article model.

Our Article model is now ready for testing and we'll be making integration tests
but before that let's make a factory.

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

Create an integration test for the Article model at
*tests/Integration/models/ArticleTest.php* by running:

{% highlight bash %}
touch tests/Integration/models/ArticleTest.php
{% endhighlight %}

Modify its contents to look like this:

{% highlight php %}
<?php

namespace Tests\Integration;

use App\Article;
use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;

class ArticleTest extends TestCase
{
    /** @test */
    function it_fetches_most_viewed_articles()
    {
        factory(Article::class, 5)->create();
        factory(Article::class)->create(['views' => 100]);
        $mostViewed = factory(Article::class)->create(['views' => 200]);

        $articles = Article::mostViewed();

        $this->assertEquals($mostViewed->id, $articles->first()->id);
    }
}
{% endhighlight %}

Our test creates 5 articles with 0 views, another article with 100 views and the
most viewed article with 200 views.

## Running the test and adjusting the model based on the results.

Run phpunit.

{% highlight bash %}
phpunit tests/Integration/ArticleTest
{% endhighlight %}

or

{% highlight bash %}
./vendor/bin/phpunit tests/Integration/ArticleTest
{% endhighlight %}

if the command above doesn't work.

The test would fail because the *Article::mostViewed()* method hasn't been
implemented. Let's implement the *Article::mostViewed()* method by modifying the
Article model to look like this:

{% highlight php %}
<?php
namespace App;

use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    public function scopeMostViewed($query)
    {
        $query->orderBy('views','desc');
    }
}
{% endhighlight %}

If you run *phpunit* now it would still fail because the data from the last test
persisted. We need to refresh the database everytime the test is executed. We
refresh it by using **RefreshDatabase** from *"Illuminate\Foundation\Testing*".
ArticleTest should look like this:

{% highlight php %}
<?php

namespace Tests\Integration;

use App\Article;
use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;

class ArticleTest extends TestCase
{
    use RefreshDatabase;

    /** @test */
    function it_fetches_most_viewed_articles()
    {
        factory(Article::class, 5)->create();
        factory(Article::class)->create(['views' => 100]);
        $mostViewed = factory(Article::class)->create(['views' => 200]);

        $articles = Article::mostViewed();

        $this->assertEquals($mostViewed->id, $articles->first()->id);
    }
}
{% endhighlight %}

If you run phpunit the test would succeed.
