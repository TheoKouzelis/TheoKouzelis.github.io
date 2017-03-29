---
layout: post
title: "AngularJS SEO"
excerpt: "My experiences with Google indexing an AngularJS website."
date: 2015-03-29 22:40:00
categories: javascript
tags: [javascript, seo, angularjs]
---
A month after releasing a new AngularJS site I started reviewing how Google had indexed the site. To my surprise
Google hadn't done such a bad job. Google had announced last year that they can now [index sites that rely on 
JavaScript](http://googlewebmastercentral.blogspot.co.uk/2014/05/understanding-web-pages-better.html) and this seems 
to work 70% of the time. But some pages weren't rendering, leaving AngualrJS variables in the page titles and 
descriptions. With one of the pages being the home page, we needed to find a solution.  

We looked into implementing [Googles AJAX crawling scheme](https://developers.google.com/webmasters/ajax-crawling/docs/getting-started)
using the [prerender/prerender](https://github.com/prerender/prerender) package. We hesitated when we discovered 
a blog post saying that [Google might stop crawling sites in this method](http://searchengineland.com/google-may-discontinue-ajax-crawlable-guidelines-216119).
The team decided to continue with prerender hoping it would fix other search engines if Google had dropped support.  

Adding prerender to our setup was easy and re-fetching the Google bot fixed the indexes with errors. I can definitely say Google is still using the AJAX crawling scheme and I would recommend prerender to anyone trying to reliably index a JavaScript heavy site.  
