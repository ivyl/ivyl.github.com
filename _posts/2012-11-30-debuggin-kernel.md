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
just like you used to do with your desktop applications...

As it appears, some twisted fairy already fulfilled that wish. The thing is
called KGDB and  it is both a little scary and pretty awesome.

## KGDB

[KGDB](http://kgdb.linsyssoft.com/), as it's name suggests, uses GDB, and
does that in all it's glory. You can peek at variables, execute code, jump
around, create breakpoints (even conditional ones) peek at threads, etc. All
this while having corresponding source in front of you.

Luckily for us KGDB in a light version was merged around mid-2.6 series into
mainline. That means no patching. We just must have kernel with few
options turned on (in most distributions, if not all, it means recompilation) and
need serial port connection between two PCs (virtual machine with emulated
RS232 is fine).

## Kernel

You need to build kernel on target machine with following options:
{% highlight text %}
CONFIG_MAGIC_SYSRQ=y
CONFIG_DEBUG_KERNEL=y
CONFIG_DEBUG_INFO=y
CONFIG_FRAME_POINTER=y
CONFIG_KGDB=y
CONFIG_KGDB_SERIAL_CONSOLE=y
{% endhighlight %}

Note that those are essentials and there are lots of other DEBUG options.
They are in form of:

{% highlight text %}
CONFIG_DEBUG_NAME
{% endhighlight %}

Check them out in *Kernel Hacking* configuration section.

If you want to step into not your code you should also turn off GCC
optimizations (changing **-O2** flag to **-O0**) on parts of the kernel of
your interest. Otherwise be prepared to strange behaviour when viewing code.
There will be optimized out values, odd code jumping, etc. Your module also
should be compiled with **-O0**, **-g** and **-ggdb** options to add in
symbols and turn off optimizations.


Remember - do not ever turn optimizations on the whole kernel. Developers
used some wicked wizardry like depending on function in-lining that
comes with **-O2** flag. Out in the intertubes exists
[patch](http://code.google.com/p/kgtp/downloads/detail?name=co.patch) that
aims stripping optimizations from most of the Linux, excluding parts that
needs them. You may be interested, I haven't found it usable.


## Setup

On the other end of serial connection, in order to run GDB, you will need
**vmlinux** file (it is uncompressed and contains debugging symbols) which
you should find in root of kernel build directory. The file should weight
around 100MiB. Additionally you have to keep kernel and module sources around
since it will be handy to have GDB print them along.


### RS232

If you physically connected two machines then you are ready with **ttyS0**.
If you used VM's virtual COM then you should get socket somewhere in file
system. Check configuration for specific path. You can use **socat** to turn
it into char device.

{% highlight bash %}
socat -d -d /home/ivyl/virtualbox/myvm/serial1 pty
{% endhighlight %}

It will print out device path it created.

### Target Machine

We need to prepare target (the one to be debugged) system. KGDB needs to know
on which device it's supposed to listen and which baudrate it should use. You
can do it via **/sys**:

{% highlight bash %}
echo ttyS0,115200 > /sys/module/kgdboc/parameters/kgdboc
{% endhighlight %}

or by adding kernel parameter:
{% highlight text %}
kgdboc=ttyS0,115200 
{% endhighlight %}

If you are using virutalized com ttyS0 should be fine.

You may also be interested in **kgdbwait** parameter. It will make kernel
to break as soon as possible during boot process.

Now only thing left is to break the machine (stop execution, wait for
debugger interaction) by pressing SysRq-G (where SysRq is Alt + PrtSc) or
executing from command line the following:

{% highlight bash %}
echo g > /proc/sysrq-trigger
{% endhighlight %}

### Firing GDB

With above preparations done, you have to attach debugger to it. I recommend
doing it in kernel's source directory. It's easiest way of making GDB source
aware.

{% highlight bash %}
cd /usr/src/linux-src
gdb /path/to/copied/vmlinux
{% endhighlight %}

Now you are within GDB shell. You can connect to remote machine by setting
same baudrate and point device you want connect to.

{% highlight text %}
set remotebaud 115200
target remote /dev/pts/4
{% endhighlight %}

You should substitude **/dev/pts/4** with device of your choice (one created by
socat or the physical).

If you broke target system you should be already able to poke around. If not,
look above for advice how to do it. 

If you don't know what to do now take some example [GDB
tutorial](http://www.cs.cmu.edu/~gilpin/tutorial/). There is much more to
GDB. You may also want to try GUI called
[DDD](http://www.gnu.org/software/ddd/).


### And Module?

That's fine, but how to debug mentioned modules, you ask? Nothing simpler! If
you are testing own module, in it's build directory, after compilation with
appropriate flags, there should be two files that matters to us.
**module.ko**, which you will load through insmod, and **module.o** which we
will load into GDB since it contains symbols.

Just load .o on the target machine. Now take a look at file:

{% highlight text %}
/sys/module/<module_name>/sections/.text
{% endhighlight %}

It contains memory address where module was loaded. Now, on the other
machine, you should feed GDB with **module.o** and mentioned address using:

{% highlight text %}
add-symbol-file module.o 0xd80a4400
{% endhighlight %}


Voila! Now go debug like there's no tomorrow.

**PS.** I also made [this script](https://gist.github.com/4171173) to make
repetitive task around KGDB automated. It's not all pretty and shiny but
maybe you will find it useful.
