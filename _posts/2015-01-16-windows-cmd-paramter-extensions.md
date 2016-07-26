---
layout: post
title:  "Windows CMD Parameter Extensions"
date:   2015-01-16 22:31:49
categories: scripts
tags: [cmd]
description: Parameter extensions for windows batch files.
---

> Parameters
> ----------
>
> A parameter (or argument) is any value passed into a batch script:
>
> C:> MyScript.cmd January 1234 "Some value"
>
> Parameters may also be passed to a subroutine with CALL:
>
> CALL :my_sub 2468

> You can get the value of any parameter using a % followed by it's numerical
> position on the command line. The first item passed is always %1 the second
> item is always %2 and so on
>
> %* in a batch script refers to all the arguments (e.g. %1 %2 %3 %4 %5
> ...%255)
>
> Parameter Extensions
> --------------------
>
> When a parameter is used to supply a filename then the following extended
> syntax can be applied:
>
> we are using the variable %1 (but this works for any parameter)
>
> * %~f1 Expand %1 to a Fully qualified path name - C:\utils\MyFile.txt
>
> * %~d1 Expand %1 to a Drive letter only - C:
>
> * %~p1 Expand %1 to a Path only e.g. \utils\ this includes a trailing \ which
> may be interpreted as an escape character by some commands.
>
> * %~n1 Expand %1 to a file Name without file extension C:\utils\MyFile or if
> only a path is present (with no trailing backslash\) - the last folder in
> that path.
>
> * %~x1 Expand %1 to a file eXtension only - .txt
>
> * %~s1 Change the meaning of f, n, s and x to reference the Short 8.3 name
> (if it exists.)
>
> * %~1   Expand %1 removing any surrounding quotes (")
>
> * %~a1 Display the file attributes of %1
>
> * %~t1 Display the date/time of %1
>
> * %~z1 Display the file size of %1
>
> * %~$PATH:1 Search the PATH environment variable and expand %1 to the fully
> qualified name of the first match found.
>
> The modifiers above can be combined:
>
> * %~dp1 Expand %1 to a drive letter and path only
>
> * %~sp1 Expand %1 to a path shortened to 8.3 characters
>
> * %~nx2 Expand %2 to a file name and extension only
>
> These parameter/ argument variables are always denoted with a single leading
> % This is unlike regular variables which have both leading and trailing %'s
> such as %variable%, or FOR command variables which use a single leading % on
> the command line or a double leading %% when used in a batch file.

Source: [http://ss64.com/][source].

[source]: http://ss64.com/nt/syntax-args.html
