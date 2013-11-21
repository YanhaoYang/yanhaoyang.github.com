---
layout: post
title: "Debug a deadlock caused by the ruby JSON parser Oj"
date: 2013-11-21 08:01
comments: true
categories:
---

Recently some busy processes hang sometimes on a production server,
where an application based Rails 3.2.13 is running on Ruby 1.9.3-p448.

The problem happened randomly, making it very difficult to debug. Basically,
the processes sent tens of requests to another server to get some JSON documents,
then parsed the JSON documents with Oj and initialize many nested objects with
that data. The processes usually hang after sending a few tens of requests.

After some research, I found some tools to help me locate the problem.

## strace

Suppose the process 1234 has hang, following command will show where the process
hang:

```
strace -p 1234
```

However, it seems not quit useful because the information is very simple.

If I run the command before the process hung, I can roughly see which part
of the Ruby code is running.

## crash-watch

`crash-watch` is a [gem](https://github.com/FooBarWidget/crash-watch) which
"monitor processes and display useful information when they crash".

This is much more useful than strace in this case. It prints out the whole
c call stacks. Now I see that the process stopped after a call to `pthread_mutex_lock`,
which is called from `mark` function in val_stack.c file, line 39.

```
#0  0x00007fbb8058d89c in __lll_lock_wait () from /lib/x86_64-linux-gnu/libpthread.so.0
#1  0x00007fbb80589080 in _L_lock_903 () from /lib/x86_64-linux-gnu/libpthread.so.0
#2  0x00007fbb80588f19 in pthread_mutex_lock () from /lib/x86_64-linux-gnu/libpthread.so.0
#3  0x00007fbb756b3d66 in mark (ptr=<optimized out>) at val_stack.c:39
#4  0x000000000041fcf9 in gc_mark_children (ptr=135392480, objspace=0x25da8a0) at gc.c:1969
#5  gc_mark_stacked_objects (objspace=0x25da8a0) at gc.c:1506
#6  gc_marks (objspace=0x25da8a0) at gc.c:2578
#7  0x00000000004207ea in garbage_collect (objspace=0x25da8a0) at gc.c:2602
#8  0x0000000000420e68 in garbage_collect_with_gvl (objspace=0x25da8a0) at gc.c:731
#9  vm_malloc_prepare (size=129, objspace=0x25da8a0) at gc.c:761
#10 vm_xmalloc (objspace=0x25da8a0, size=<optimized out>) at gc.c:795
#11 0x00000000004c9ec3 in rb_str_buf_new (capa=128) at string.c:745
...
```

When I found the source code in Oj gem, it has been clear that the problem is a
deadlock cause by the gem.

## nrdebug

After that, I found a more powerful tool built by New Relic: `nrdebug` in `bin` directory of
the newrelic_rpm gem. It collects everything about the process thoroughly, including environment
variables, c backtraces, ruby backtraces, and open files.

I should have known it earlier.


