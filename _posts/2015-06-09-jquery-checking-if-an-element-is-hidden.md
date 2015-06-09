---
layout: post
title: "jquery: Checking if an element is hidden"
date:   2015-06-09 08:31:49
categories: ["js", "jquery"]
---

Checking if an element is hidden:

{% highlight javascript %}
$(element).is(":visible");
{% endhighlight %}

This solution encourages the confustion of `visible=false` and `display:none`.