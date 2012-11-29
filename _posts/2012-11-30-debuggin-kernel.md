---
layout: post
title: "Debugging Kernel with KGDB"
tags: [kernel]
---
{% include JB/setup %}

## The Problem

Suppose you've written kernel module and it do not work as intended. You
cannot find anything by reading code. **printk** debugging left you with noting.
You wish there was a way to look how things act in the wild... It would be
nice to run some debugger, create few breakpoints and operate on live data,
just like you used to do with your desktop applications.

As it appears, some twisted fairy already granted that wish and made it
happen. The thing is called KGDB and despite being a little scary is pretty
awesome.

## KGDB

KGDB, as it's name suggests, uses GDB, and does that in all it's glory. You
can peek at variables, execute code, jump around, create breakpoints (even
conditional ones) peek at threads, etc. All this while having corresponding
source in front of you.

Luckily for us KGDB in light version was merged around mid-2.6 series into
mainline, so no patching needed. We just have to have kernel with few
options turned on (in most distributions, if not all, it means recompilation) and
need serial port connection between two PCs (virtual machine with emulated
RS232 is fine).

## Kernel

You need to build kernel on target machine with following options (which are
essential for it to be usable):
{% highlight text %}
CONFIG_MAGIC_SYSRQ=y
CONFIG_DEBUG_KERNEL=y
CONFIG_DEBUG_INFO=y
CONFIG_FRAME_POINTER=y
CONFIG_KGDB=y
CONFIG_KGDB_SERIAL_CONSOLE=y
{% endhighlight %}

Note that there are lots of DEBUG options for most subsystem. They are in form
of:

{% highlight text %}
CONFIG_DEBUG_NAME
{% endhighlight %}

You might be interested in some of them.

You should also turn off GCC optimization flag (changing **-O2** to **-O0**)
on parts of the kernel you want to debug (i.e. those that are incident to
code you want test). Your module also should be compiled with **-O0**, **-g**
and **-ggdb** options to include all needed debugging symbols.


Remember - do not ever turn optimization on the whole kernel. Developers
made some wicked optimizations like depending on function in-lining that
comes with **-O2** flag. Out in the intertubes exists
[patch](http://code.google.com/p/kgtp/downloads/detail?name=co.patch) that
aimed stripping optimizations from most of the kernel, excluding parts that
needs them. You may be interested. I haven't found it usable.


## Setup

To setup GDB you will need **vmlinux** file (it is uncompressed and contains
all the debugging symbols) which you should find in root of kernel build
directory. It should weights above 100MiB (symbols are heavy). Additionally
you should keep kernel and module sources around since GDB will look for
them.

In module directory, after compilation, there should be two files that
matters to us. **module.ko**, which we load through insmod, and
**module.o** which we will load into GDB.

### RS232

If you physically connected two machines then you are ready with **ttyS0**.
If you used VM's virtual COM then you should get socket somewhere in file
system. You can use **socat** to turn it into char device.

{% highlight bash %}
socat -d -d /home/ivyl/virtualbox/myvm/serial1 pty
{% endhighlight %}

It will print out device path it created.

### Target Machine
We need to prepare target (the one to be debugged) system. KGDB needs to know
on which device it's supposed to listen and which baud rate to use. You can
do it via **/sys**:
{% highlight bash %}
echo ttyS0,115200 > /sys/module/kgdboc/parameters/kgdboc
{% endhighlight %}

or by adding kernel parameter:
{% highlight text %}
kgdboc=ttyS0,115200 
{% endhighlight %}

You may also be interested in **kgdbwait** parameter. It will make kernel
to break as soon as possible during boot process.

You can also break the machine (stop execution, wait for debugger interaction) by
pressing SysRq-G (where SysRq is Alt + PrtSc) or executing from command line
the following:

{% highlight bash %}
echo g > /proc/sysrq-trigger
{% endhighlight %}

### Firing GDB

Now you can run GDB. I recommend doing it in kernel's source directory.
{% highlight bash %}
gdb vmlinux
{% endhighlight %}

Now you are within GDB shell. You can connect to remote machine by setting
same baudrate and point device you want connect to.

{% highlight text %}
set remotebaud 115200
target remote /dev/pts/4
{% endhighlight %}

If you broke target system you should be already able to poke around. If not,
look above for advice how to do it.

### And Module?

That's fine, but how to debug modules, you ask? Nothing simpler! Just
load it on the target machine. Now take a look at file:

{% highlight text %}
/sys/module/<module_name>/sections/.text
{% endhighlight %}

It contains memory address where module was loaded. Now you should feed GDB
with **module.o** and mentioned address using:

{% highlight text %}
add-symbol-file module.o 0xd80a4400
{% endhighlight %}


Voila! Now you can debug like there's no tomorrow.

**PS.** I also made [this script](https://gist.github.com/4171173) to make
repetitive task around KGDB automated. It's not all pretty and shiny but
maybe you could find it useful.
