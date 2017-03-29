---
layout: post
title: "Removing Laravel Environment Variables From Airbrake Notices"
excerpt: "Without configuration, Airbrake's PHP package can gather sensitive data. Here is how to stop Laravel's environment variables ending up in your logs."
date: 2016-03-13 01:00:00
categories: php
tags: [php, laravel, logging]
---
When I first started using Airbrake I was impressed by the amount of detail gathered by each tab of the notice. But when I started to 
inspect the "env" tab I notice that sensitive data such as database passwords where being logged from my Laravel project.

This occurs because Laravel uses a package called [DotEnv](https://github.com/vlucas/phpdotenv) which loads variables set in the project's .env file into
the [$_SERVER super global](https://github.com/vlucas/phpdotenv/blob/master/src/Loader.php#L341). Each time the Airbrake package builds a notice, the super globals [$_SESSION, $_REQUEST and $_SERVER](https://github.com/airbrake/phpbrake/blob/master/src/Notifier.php#L107,L119) are copied
and sent along with the error message and context data.

To stop sensitive data ending up in the logs, Airbrake's API has a method called "filter" which allows you to add callbacks which can edit
each error notice before it is sent. To automate the removal of environment variables from the logs I have created a [Laravel service provider for Airbrake](https://github.com/TheoKouzelis/laravel-airbrake).

The service provider will configure an instance of Airbrake\Notifier with an ID, key and environment name. The service provider will also register a filter
that loops through unseting variables set in the .env file from notices. Any variable set in the .env file will appear with a value of "FILTERED" in the 
env tab of an Airbrake report.
