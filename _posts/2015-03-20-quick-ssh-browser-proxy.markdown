---
layout: post
title: "Quick SSH Browser Proxy"
description: "Quick SSH proxy to test your websites geo features in your browser"
date: 2015-03-20 21:00:00
categories: command-line
---
My boss showed me a great little trick today which has helped me view our website from different 
locations around the world. There are lots of third party services which can help you test your site 
from different locations but if you have SSH access to a server in the desired location you may not 
need them. At work we use AWS so we can very quickly have a server up America, Europe and Asia.   

In Firefox open up the preferences and select the network tab, then press the connections settings button.  
![alt text](https://theo.codes/images/network.png "Firefox preferences network tab")   
Then in connections settings select manual proxy configuration. Then type localhost for your SOCKS Host and 
then a random port number e.g 4321.   
![alt text](https://theo.codes/images/proxy.png "Firefox preferences proxy settings")   

Then go to the command line and SSH into your remote server using the bind address port option -D with 
you SOCKS host port.   

ssh -D 4321 username@server-ip

Now when you continue to browse in Firefox you will be doing so from your remote server. Now you can 
test your geo features from a different location.

