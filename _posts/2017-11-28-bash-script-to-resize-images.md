---
layout: post
title: "bash script to resize multiple files"
date:   2017-11-28 22:31:49
categories: ["bash", "find", "oneliners"]
Description: A bash command to resize multiple files recursive in a directory using find and convert.
---

This is a small bash script to decrease resolution of JPG photo or other image
files. It searches all `.JPG` files recursively from the current directory `.`
and the converts them to `1200` pixel wide. It uses ImageMagick's convert tool,
so you will need `imagemagick` package to execute this command. By changing the
search pattern and the expected resolution you can customize it to you needs...

{% highlight bash %}
find . -iname "*.JPG" | while read f; do convert -resize 1200 "$f" "$f"; done
{% endhighlight %}

