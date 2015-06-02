---
layout: post
title:  "Delete a Git branch both locally and remotely"
date:   2015-06-02 08:31:49
categories: ["git"]
---

To delete a branch locally:

{% highlight bash %}
git branch -D <branchName>
{% endhighlight %}

And to remove the remote branch:

{% highlight bash %}
git push origin --delete <branchName> vagy git push origin :<branchName>
{% endhighlight %}