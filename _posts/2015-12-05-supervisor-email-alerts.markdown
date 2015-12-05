---
layout: post
title: "Supervisor Email Alerts"
description: "How to getting email alerts when Supervisor can't restart your program"
date: 2015-12-05 01:00:00
categories: linux
---  
[Supervisor](http://supervisord.org/) is a great python application that will make sure that all your long running programs on the server stay running. 
I was first introduced to this tool through the [Laravel framework](http://laravel.com/) which requires Supervisor to keep its command line program 
running to process events that have been pushed on to a [queue](http://laravel.com/docs/5.1/queues#supervisor-configuration).  

This combination was working great until one day we notice the queue was no longer being processed. Checking the server we 
found our program had entered the following status [“FATAL Exited too quickly”](http://supervisord.org/subprocess.html#process-states). By default Supervisor will try to restart your 
program 3 times before deciding to permanently give up and assign a status of FATAL to that program.  

You can set Supervisor to permantly to restart your program, but in our case if the program had failed 3 times it's probably 
due to an issue that we need to be alerted to. Fortunately Supervisor allows you to add [listeners to events](http://supervisord.org/events.html) like program 
status changes. Even more conveniently there is already a suit of listeners called [Superlance](http://superlance.readthedocs.org/en/latest/index.html#)
which handle common tasks like email failed program alerts.  

After installing Superlance via pip  

{% highlight php %}
pip install superlance
{% endhighlight %}

We used the [fatalmailbatch](http://superlance.readthedocs.org/en/latest/fatalmailbatch.html) listener to email the team when a program entered a FATAL state. Adding a listener is just like 
adding any other program. I added a new config file at /etc/supervisor/conf.d/fatalmailbatch.conf  

{% highlight php %}
# /etc/supervisor/conf.d/fatalmailbatch.conf 

[eventlistener:fatalmailbatch]
command=fatalmailbatch 
  --toEmail="admin@team.com" 
  --fromEmail="supervisor@app.com" 
  --smtpHost="smtp.mailgun.org" 
  --userName="username" 
  --password="password"
events=PROCESS_STATE,TICK_60
autostart=true
autorestart=true
{% endhighlight %}

And then started the listener running with supervisorctl  

{% highlight php %}
supervisorctl reread
supervisorctl add fatalmailbatch
supervisorctl start fatalmailbatch
{% endhighlight %}

All in all pretty simple and adds a bit more peace of mind.  
