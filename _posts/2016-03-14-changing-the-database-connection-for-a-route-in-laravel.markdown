---
layout: post
title: "Changing The Database Connection For A Route In Laravel"
excerpt: "How to create middlewares that change the database connection of your Laravel project."
date: 2016-03-14 01:00:00
categories: php
tags: [php, laravel]
---
While creating an API in Laravel I had a requirement due to a tight schedule. The content team required a clean database 
to start provisioning real data in the backend CMS and the app developer still required the API to serve the full range of 
fake seeded data.   

We could have created two separate installations of the same Laravel application, but I had a feeling it might be easier to 
use two databases (one clean and one seeded) and have them serve the two different route groups “backend” and “api”.

With [middlewares](https://laravel.com/docs/master/middleware) this task was very easy to do. First I created a new middleware using 
an [artisan command](https://laravel.com/docs/master/artisan).

{% highlight php %}
php artisan make:middleware UseSeededDatabase
{% endhighlight %}

In the generated middleware I use the Config facade to set the 'database.default' option to our seeded database connection.

{% highlight php %}
<?php

namespace App\Http\Middleware;

use Closure;
use Config;

class UseSeededDatabase
{
    public function handle($request, Closure $next)
    {
        Config::set('database.default', 'seededmysql');

        return $next($request);
    }
}
{% endhighlight %}

Then I added the middleware to the HTTP Kernal.

{% highlight php %}
<?php

//app/Http/Kernel.php 

protected $routeMiddleware = [
    'seeded.database' => \App\Http\Middleware\UseSeededDatabase::class,
];
{% endhighlight %}

And setup the two database connections.

{% highlight php %}
<?php

return [

    'default' => env('DB_CONNECTION', 'cleanmysql'),

    'connections' => [

        'cleanmysql' => [
            'driver'    => 'mysql',
            'host'      => 'localhost',
            'database'  => 'cleandatabase',
            'username'  => 'user',
            'password'  => 'password', 
            'charset'   => 'utf8',
            'collation' => 'utf8_unicode_ci',
            'prefix'    => '',
            'strict'    => false,
        ],

        'seededmysql' => [
            'driver'    => 'mysql',
            'host'      => 'localhost',
            'database'  => 'seededdatase', 
            'username'  => 'user',
            'password'  => 'password', 
            'charset'   => 'utf8',
            'collation' => 'utf8_unicode_ci',
            'prefix'    => '',
            'strict'    => false,
        ],
];
{% endhighlight %}

This meant by default any route that didn't use the 'seeded.database' middleware would be using 
the default 'cleanmysql' connection. And any route using the middleware would have the database connection
overridden to the 'seededmysql' connection.  

The only thing left to do is apply the middleware to the API routes.

{% highlight php %}
<?php

Route::group(['prefix' => 'api', 'middleware' => ['database.fake']], function () {
        Route::resource('resource1', 'Api\Controller');
});

Route::group(['prefix' => 'backend'], function () {
        Route::get('dashboard', 'Backend\Controller@index');
});
{% endhighlight %}

Then seed the 'seededmysql' connection.

{% highlight php %}
php artisan migrate --database="seededmysql" 
php artisan db:seed --database="seededmysql" 
{% endhighlight %}
