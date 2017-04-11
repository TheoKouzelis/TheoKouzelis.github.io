---
layout: post
title: "How to Handle the Laravel TokenMismatchException"
excerpt: "Handling the TokenMismatchExeception isn't entirely obvious. Here are two ways to easily take care of them, within your Laravel apps."
date: 2017-04-11 00:00:00
categories: php
tags: [php, laravel]
---
As part of Laravel’s CSRF protection, the framework will require you to 
post back a token when submitting a form. The token is a way of verifying 
that the logged in user is filling out a form created by the website. A 
TokenMismatchException is thrown when a form is submitted with a token that 
doesn’t match the one stored in the session.  

Like 404 error handling, CSRF protection is another out of the box benefit 
of using Laravel. But unlike 404 exceptions, where a HTML template is 
rendered, the TokenMismatchException results in the frameworks debug page.    

Problems arise when legitimate users leave forms open, for periods greater 
than the time set in session.lifetime (default 120 minutes in 
config/session.php). The session holding the token expires and will no longer 
match the token finally submitted by the form. This will result in an 
Exception being thrown and the user receiving a “Whoops, looks like something 
went wrong” page.  

Researching various solutions from the Laravel forums, the most suitable is 
to redirect the user back to the form they have submitted. The following code 
demonstrates how to do this from the frameworks global exception handler along 
with an error message.  

{% highlight php %}
<?php

//app/Exceptions/Handler.php

    public function render($request, Exception $exception)
    {
        if ($exception instanceof TokenMismatchException) {
            return redirect()
                ->back()
                ->withErrors(["The form has expired due to inactivity. Please try again."]);
        }

        return parent::render($request, $exception);
    }

{% endhighlight %}

Some solutions assume if the session has expired, the user will be logged out 
and they should be redirected to the login screen. But if the user has checked 
the “remember me” box during login, they will continue to be logged in across 
individual sessions. This also doesn’t account for forms on public 
pages. So simply returning the user back to the form is the best course of action.

An additional solution is to install a package called 
[Laravel Caffeine](https://github.com/GeneaLabs/laravel-caffeine). This 
package keeps your session from expiring through a middleware which adds a 
script to poll your app via ajax. The activity from the ajax requests will 
stop your sessions from expiring and forms left open will be able to submit 
without exception. The [middleware](https://github.com/GeneaLabs/laravel-caffeine/blob/master/src/Http/Middleware/LaravelCaffeineDripMiddleware.php) 
cleverly only adds the script to pages that contain a form.   

But, stopping sessions from expiring, could be seen as a downgrade of the 
overall security of a session.    
