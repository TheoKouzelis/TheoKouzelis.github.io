---
layout: post
title: "Proposed Change to Laravel's Service Container"
description: "A change to Laravel's Service Container which will allow contextual Monolog configurations"
date: 2016-10-26 01:00:00
categories: php
---
There is a [open pull request](https://github.com/laravel/framework/pull/15637) by 
[Jordan Pittman](https://github.com/thecrypticace) to amend the Laravel's Service Container to allow contextual 
bindings for bindings registered as singletons.  

My personal use case is to allow different configured loggers to be injected into different parts of your application.  

In the early stages of the frameworks bootstrap process. 
[Laravel creates an instance of Monolog](https://github.com/laravel/framework/blob/5.3/src/Illuminate/Foundation/Bootstrap/ConfigureLogging.php#L41) 
and binds it to the alias ‘log’. This instance can be configured using [a custom function or one of Laravel's presets](https://laravel.com/docs/5.3/errors#configuration). 
Any time your application depends on ‘log’ or one of its [orthonyms (e.g Psr\Log\LoggerInterface)](https://github.com/laravel/framework/blob/5.3/src/Illuminate/Foundation/Application.php#L1093) 
this instance is served.  

With the proposed change it would be possible to override this instance for any class using the container to inject a 
logger. By creating a contextual binding we could increase the logging level of a particular class or redirect log messages 
to a different recipient without affecting the rest of the application. Deleting the contextual binding for the class would 
then see it resume with the standard logger.  

I believe this pull request is currently being reviewed.

##Example LoggerInterface Contextual Binding

{% highlight php %}
<?php

namespace App;

class Example 
{
    protected $log;

    public function __construct(\Psr\Log\LoggerInterface $log)
    {
        $this->log = $log;
    }
}
{% endhighlight %}
  

{% highlight php %}
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Monolog\Logger;
use Monolog\Handler\StreamHandler;

class AppServiceProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->when('App\Example')
            ->needs('Psr\Log\LoggerInterface')
            ->give(function () {
                $log = new Logger('example');
                $log->pushHandler(new StreamHandler(storage_path('logs/test1.log'), Logger::DEBUG));
                return $log;
            });
    }
}
{% endhighlight %}
