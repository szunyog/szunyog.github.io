---
layout: post
title:  "Vim automatically set paste mode"
date:   2016-08-15 10:31:49
categories: vim
tags: [vim]
description: Sample settings of vimrc to set paste mode automatically.
---

Vim automatically tires to indent pasted text when running inside a terminal. To
avoid this 'paste' mode can be set and unset before and after the paste
operation, by the following commands:

{% highlight vim %}
:set paste

# paste from clipboard

:set nopaste
{% endhighlight %}


Toggle key
----------

To speed up this process it is possible to specify a key sequence that toggles
the 'paste' mode.

{% highlight vim %}
set pastetoggle=<F2>
{% endhighlight %}

Now you hit F2 key to toggle the paste before and after pasting.

Automatically detect paste
--------------------------

Here's a little trick from [Coderwall][coderwall], that uses terminal's
bracketed paste mode to automatically set/unset Vim's 'paste' mode.
The following lines should be added to .vimrc:

{% highlight vim %}
let &t_SI .= "\<Esc>[?2004h"
let &t_EI .= "\<Esc>[?2004l"

inoremap <special> <expr> <Esc>[200~ XTermPasteBegin()

function! XTermPasteBegin()
  set pastetoggle=<Esc>[201~
  set paste
  return ""
endfunction

{% endhighlight %}

Under tmux
----------

If Vim is inside of a Tmux session then double escape is required.
The above config for Tmux users looks like this:

{% highlight vim %}
function! WrapForTmux(s)
  if !exists('$TMUX')
    return a:s
  endif

  let tmux_start = "\<Esc>Ptmux;"
  let tmux_end = "\<Esc>\\"

  return tmux_start . substitute(a:s, "\<Esc>", "\<Esc>\<Esc>", 'g') . tmux_end
endfunction

let &t_SI .= WrapForTmux("\<Esc>[?2004h")
let &t_EI .= WrapForTmux("\<Esc>[?2004l")

function! XTermPasteBegin()
  set pastetoggle=<Esc>[201~
  set paste
  return ""
endfunction

inoremap <special> <expr> <Esc>[200~ XTermPasteBegin()
{% endhighlight %}

[pastetoggle]: http://vim.wikia.com/wiki/Toggle_auto-indenting_for_code_paste
[coderwall]: https://coderwall.com/p/if9mda/automatically-set-paste-mode-in-vim-when-pasting-in-insert-mode 
