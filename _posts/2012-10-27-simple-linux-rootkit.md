---
layout: post
title: "Simple Linux Rootkit"
tags: [linux, kernel, c]
---
{% include JB/setup %}

[Code](https://github.com/ivyl/rootkit), [paper](http://issuu.com/ivyl/docs/rootkit) (in polish).

### About
Some time ago along with friend I wrote Linux rootkit which was ment to be as
simple as possible. It does few things:

 * creates /proc/rtkit which is used to issue commands,
 * gives root when asked,
 * hides processes and files with special names,
 * makes itself invisible.

So, how it's done?

### File hiding
File hiding (and thus processes hiding, since they are represented as files)
is achieved by hooking to filesystems (procfs and the one used at /) and
wrapping *readdir* function call. It supplies original one with our own version
of *filldir_t* which skips files we are interested in hiding.

You can still access those files, since we skip them when listing directory.
FS's lookup call is able to find them.


### Page rights
The tricky part is managing memory pages rights. Usually pointer to fs calls
sits on readonly page, so it have to be made writable. I wrote few helper
functions that looks up address and give us structure representing page. From
there we can change rights.

### Hiding presence
Module is hidden by removing it from modules list (we get pointer to linked
list element with *THIS_MODULE*), then few pointer assignments and we are done.
We also remove its kobject representation from list (/sys) using appropriate
functions.

### Commanding rootkit
We simply create procfs entry and supply it with our own read and write
functions. When reading from entry we give current status, when
writing, we issue commands. Entry is hidden in same manner as other files.



### Code
Since it's too long to include it in-line on blog,
[here's](https://gist.github.com/3964594#file_rt.c) full source code with
additional commentary.


Currently I am working on project related to Linux's VFS. Expect in-depth
tour. There will be blood, action and a lot of C code.
