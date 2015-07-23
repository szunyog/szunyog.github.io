---
layout: post
title:  "Screen"
date:   2015-06-16 08:31:49
categories: ["linux"]
---

I am using Linux for several years, but just started with Screen. From my personal view, this is a magical tool to:

* use your Linux from remote shell and like using more than one shell;
* using instable network connection and ssh link randomly closes;
* leave running processes after your connection is closed;

It is a good chance to have Screen in your Linux distribution's package library so installation is quite simple like:

{% highlight bash %}
# apt-get install screen (On Debian based Systems)
{% endhighlight %}

{% highlight bash %}
# yum install screen (On RedHat based Systems)
{% endhighlight %}

To start it just type:

{% highlight bash %}
screen
{% endhighlight %}

Or if you want multiple session you can name your current one:

{% highlight bash %}
screen -S sessionname
{% endhighlight %}

When screen started, you can do all your work as you are in the normal shell environment. But since the screen is an application, so it have command or parameters. With `Control-a ?` key combination the help screen is appeared.


If your connection had broken, but you can reconnect to get your previous session type:

{% highlight bash %}
screen -rd
{% endhighlight %}

Here is the list of the most commonly used commands and a simple startup file:

* `Control-a ?`: displays the list of the commands
* `Control-a c`: creates a new interactive shell
* `Control-a number`: switches between shells, number should be 1,2,3..
* `Control-a Shift-a`: renames a shell
* `Control-a "`: lists all the shell
* `Control-a p`: switches to the previous shell
* `Control-a n`: switches to the next shell
* `Control-a d`: detaches the screen console
* `screen -r`: restores the screen
* `Control-a w` display the screen list in the status bar

Put the content of this file into your ~/.screenrc:

{% highlight bash %}
startup_message off
#vbell off
escape /
defscrollback 5000
termcapinfo rxvt-unicode ti@:te@     #enable SHIFT-PGUP / SHIFT-PGDOWN scroll
terminfo rxvt-unicode ti@:te@:
term screen-256color
setenv LC_CTYPE en_US.UTF-8
defutf8 on
nonblock on
#vbell off
msgwait 5
{% endhighlight %}

