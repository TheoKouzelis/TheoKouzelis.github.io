---
layout: post
title: "Laravel Logout Caching Issue"
excerpt: "How to stop browsers caching pages that require authorisation in Laravel."
date: 2017-03-24 01:00:00
categories: php
tags: [php, laravel]
---
This is an issue which I'm sure effects many frameworks and custom sites. Because I work mainly with 
the Laravel, I'm using the framework to illustrate this issue. The video below demonstrates how 
browsers can cache pages that require authorisation. The cached pages can sit in a browser's history 
long after the user has logged out. This could allow unauthorized users to use the browser's back 
button to view private pages.  

{:style="text-align:center"}
[![Laravel Logout Caching Issue Video](/images/laravel-logout-thumb.png)](https://youtu.be/8CNGwOGemuM)

After a bit of Googling, I found an excellent [post by David Beitey](https://davidjb.com/blog/2011/03/disabling-caching-for-sensitive-web-pages-aka-how-to-prevent-logged-out-users-going-back/). In
this post, he outlines the required headers, to tell a browser not to cache a page. Inspired by this post I created the following middleware. When
this middleware is added to pages that require authorisation it will stop browsers caching the sensitive pages.

{% highlight php %}
<?php

namespace App\Http\Middleware;

use Carbon\Carbon;
use Closure;

class PrivateResponse
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        $response = $next($request);

        $response->withHeaders([
            'Cache-Control' => 'no-store, no-cache, max-age=0, must-revalidate, private',
            'Expires'       => Carbon::now()->format('D, d M Y H:i:s T'),
        ]);

        return $response;
    }
}
{% endhighlight %}
