---
layout: post
title: "Quick SSH Browser Proxy"
description: "Quick SSH proxy to test your websites geo features in your browser"
date: 2015-03-20 21:00:00
categories: command-line
---
My boss showed me a great trick today which has helped me view our website from different locations around the world. There are lots of third party services which can help you test your site from different locations. But if you have SSH access to a server in the desired location you may not need them. At work we use AWS, so we can have a servers up in America, Europe and Asia within minutes to test the geo features of our site.  

In Firefox open up the preferences and select the network tab, then press the connections settings button.   

![Firefox preferences network tab](https://theo.codes/images/network.png "Firefox preferences network tab")  

In the connections settings select "manual proxy configuration". Then type "localhost" for your SOCKS Host and enter a random port number e.g 4321.  

![Firefox preferences proxy settings](https://theo.codes/images/proxy.png "Firefox preferences proxy settings")

Switch over to the command line and SSH into your remote server using the bind address port option "-D" with you SOCKS host port.   

ssh -D 4321 username@server-ip   

Now when you continue to browse in Firefox you will be doing so from your remote server. Now you can test your websites geo sensitive features from as many locations as you have SSH access to.   

