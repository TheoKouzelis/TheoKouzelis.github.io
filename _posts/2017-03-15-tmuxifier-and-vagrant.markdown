---
layout: post
title: "Tmuxifier and Vagrant"
description: "Simple template to get Tmuxifier windows to login to Vagrant"
date: 2017-03-15 01:00:00
categories: linux
---
Recently I have become addicted to using [Tmux](https://tmux.github.io/), to build all my windows that make up my 
development environment. Tmux was recommended to me as a easy way to run a program in the background. But I quickly 
became reliant on it to run multiple windows, tailing logs, running task runners and vim.  

[Tmuxifier](https://github.com/jimeh/tmuxifier) is a great way to template Tmux windows and sessions. I work a lot 
with [Vagrant](https://www.vagrantup.com/) and require a few of my Tmux windows to login to the virtual machine. I 
found this hard to template at first, because the Vagrant dependant windows in a session, were always launched before 
Vagrant had booted a virtual machine.  

I finally solved this problem by creating the following Tmuxifier window, which will run “vagrant up” before loading all 
of the Vagrant dependant windows. Finally the main window exits leaving only the logged in Vagrant windows.

{% highlight php %}
new_window "vagrant-boot"

run_cmd "vagrant up"
run_cmd "tmuxifier load-window vagrant-logs"
run_cmd "tmuxifier load-window vagrant-console"
run_cmd "tmuxifier load-window vagrant-gulp"
run_cmd "tmux kill-window"
{% endhighlight %}
