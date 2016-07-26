---
layout: post
title:  "Git undo a commit and redo"
date:   2015-05-01 10:31:49
categories: git
description: How to undo a git commit.
---

When you need to undo your last git commit...

{% highlight bash %}

$ git commit ...              (1)
$ git reset --soft HEAD~1     (2)
<< make corrections >> (3)
$ git add ....                (4)
$ git commit -c ORIG_HEAD     (5)

{% endhighlight %}

1. The last commit what should be undone.
2. Reset to the parent "commit", "soft" option does not touch the index file or the working tree at all, but resets the head to <commit>,</commit>.
3. Make corrections to working tree files. 
4. Stage changes for commit.
5. Commit the changes.

