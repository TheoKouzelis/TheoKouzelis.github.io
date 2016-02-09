---
layout: post
title: "Speeding Up Laravel Tests"
description: "A quick way to speed up your functional tests in Laravel"
date: 2016-02-08 01:00:00
categories: php
---
While investigating ways to speed up tests by [Chris Duell](http://www.chrisduell.com/blog/development/speeding-up-unit-tests-in-php/) 
and [Jordan Eldredge](https://jordaneldredge.com/blog/speed-up-laravel-tests-with-database-transactions/). I stumbled upon an 
alternative way which might not be as fast but is easy to implement.  

If you are using a [Sqlite in-memory database](https://www.sqlite.org/inmemorydb.html) during your tests and the cause of your 
speed issues is your migrations. You can gain a 50% speed increase by avoiding the DatabaseMigrations trait and implementing 
your own.

The [DatabaseMigrations trait](https://github.com/laravel/framework/blob/5.1/src/Illuminate/Foundation/Testing/DatabaseMigrations.php)
uses the phpunit [@before](https://phpunit.de/manual/current/en/appendixes.annotations.html#appendixes.annotations.before)
annotation to execute a function before each test. The function runs the "migrate" command and adds a hook to run the 
"mirgate:rollback" command when the test is finished. But because we are using a in-memory database, the database is destroyed 
and created with every new database connection. This means we can save time by not rolling back the migrations at the end of 
each test.

To implement migrations without rollbacks you have to do the following.

Add the im-memory sqlite config to your connections array
{% highlight php %}
<?php

#config/database.php

return [
    'connections' => [
        'sqlite' => [
            'driver'   => 'sqlite',
            'database' => ':memory:',
            'prefix'   => '',
        ],
    ]
];
{% endhighlight %}

Set your tests to run against sqlite database by adding a env variable
{% highlight php %}
<?php

#tests/TestCase.php

class class TestCase extends Illuminate\Foundation\Testing\TestCase
{
    protected $baseUrl = 'http://localhost';
  
    public function createApplication()
    {
        putenv('DB_CONNECTION=sqlite');
  
        $app = require __DIR__.'/../bootstrap/app.php';

        $app->make(Illuminate\Contracts\Console\Kernel::class)->bootstrap();

        return $app;
    }
}
{% endhighlight %}

Then add the migration function to your TestCase
{% highlight php %}
<?php

#tests/TestCase.php

class class TestCase extends Illuminate\Foundation\Testing\TestCase
{
    protected $baseUrl = 'http://localhost';
 
    /**
    * @before
    */
    public function runDatabaseMigrations()
    {
        $this->artisan('migrate');
    }
  
    public function createApplication()
    {
        putenv('DB_CONNECTION=sqlite');
  
        $app = require __DIR__.'/../bootstrap/app.php';

        $app->make(Illuminate\Contracts\Console\Kernel::class)->bootstrap();

        return $app;
    }
}
{% endhighlight %}
