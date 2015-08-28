---
layout: post
title: "Setting Up Airbrake With Monolog"
description: "Quick and easy guide to sending log messages from Monolog to the external bug tracker Airbrake"
date: 2015-09-27 01:00:00
categories: php
---  
Integrating Monolog and Airbrake is fairly painless. The hardest part  is navigating a lot of deprecated packages for 
Airbrake that still rank higher in searches than the currently supported API [airbrake/phpbrake](https://github.com/airbrake/phpbrake). 
This package as well as being an API to contact Airbrake, also ships with a handler for Monolog.  

To begin, create a new project in Airbrake and obtaining a “Project ID” and a “Project API Key” from the General Settings page in the top right hand corner.  

Then install [airbrake/phpbrake](https://github.com/airbrake/phpbrake) via composer then add the following code 
to your Monolog setup:  

{% highlight php %}
<?php

use Airbrake\MonologHandler;
use Airbrake\Notifier;
use Monolog\Logger;

$logger = new Logger('app');

$airbrakeNotifier = new Notifier(array(
    'projectId' => 1234,
    'projectKey' => 'your-key',
));

$airbrakeNotifier->addFilter(function ($notice) {
    $notice['context']['environment'] = 'my-dev'; //default is production
    return $notice;
});

$logger->pushHandler(new MonologHandler($airbrakeNotifier, Logger::ERROR));

?>
{% endhighlight %}

Airbrake has a great feature that allows you to name the environments. This allows you to get used to using Airbrake
during development. Use the $notice['context']['environment'] variable during setup to dynamically name your environments.
Then under "Notification Settings" you can set Airbrake to only contact you when the environment named "production" logs
a message.

By default Airbrake will group all log messages by their Monolog error level. I found this grouping very unhelpful
as it would group completely unrelated messages as the same error. To force Airbrake into ungrouping the error levels 
you must navigate to the “General Settings” page and add the project name followed by the error level (e.g ProjectName.NOTICE) into the “Distinct grouping” textarea and press save.  

Example Distinct groupings: 

* ProjectName.NOTICE 

* ProjectName.ALERT 

* ProjectName.WARNING   

* ProjectName.ERROR    

* ProjectName.CRITICAL 


After that you should be set up and ready to log.
