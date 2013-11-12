---
layout: post
title: "SSH Time-Based One-Time Passwords"
tags: [unix, security]
---
{% include JB/setup %}

## Intro

*I don't alway log in remotely using passwords on unsafe machine, but when I
do, I make sure it's secure.*

In order to be certain my system is safe, even if my password had been
overheard/peeped/logged, I use 2-step authentication using Time-Based
One-Time Passwords. There are few implementations of this idea available.
Reading through I have chosen OATH.

I alway carry my phone, on which I have installed app that generates tokens.
I make sure that time is synced up using carrier synchronization on Android
and NTP on servers (you could use NTP on phone your carrier sucks). And now I
can log in from anywhere using current token along with my password that I
change on regular basis.

## OATH

[OATH](http://en.wikipedia.org/wiki/Initiative_For_Open_Authentication) is
open standard, with well described algorithms (as RFCs), reference
architecture is available and much more resources. That's the thing that
Amazon use.

You should install [oath-toolkit](http://www.nongnu.org/oath-toolkit/), which
is OATH implementation for unix-like systems using PAM:

{% highlight bash %}
portmaster security/oath-toolkit
{% endhighlight %}

Or however your religion requires you to.

## PAM

Now it's time to configure PAM. In <tt>/etc/pam.d/</tt> there should be files
corresponding to login method, such as <tt>sshd</tt>, <tt>su</tt>, etc. I
first had tested changes on <tt>su</tt> (or <tt>su-l</tt> on some systems,
for login onto other user) and after that finally merged them into
<tt>sshd</tt> configuration file.

I added auth module as *requisite* (see <tt>man pam.conf</tt> for details) on
top of configuration file. **On FreeBSD** I had to specify full path to the module.

{% highlight text %}
auth requisite /usr/local/lib/security/pam_oath.so usersfile=/etc/users.oath
{% endhighlight %}

**On most Linux** distributions, if oath-toolkit is installed from package
<tt>pam_oath.so</tt> should be sufficient.

From now on you will be asked for token first, and then for your password.

## The Userfile

The *usersfile* I pointed OATH module to is <tt>/etc/users.oath</tt>. Example provided in
documentation is very brief:

{% highlight text %}
HOTP root - 00
{% endhighlight %}


Too brief I would say. It took me some research (actually looking into source
code) to make sense of it.

White spaces are threated as delimiter. **First field** is auth method, **second** is
user name, **third field** would be used if we need password to be provided along
with token, since in this case PAM will ask us for password just after token,
we are okay with none (<tt>-</tt>). **Last field** is key in hexadecimal format.


I recommend using SHA1 of some random data:

{% highlight bash %}
head -c 2048 /dev/urandom | openssl sha1
{% endhighlight %}

HOTP is sequentional 6 digit key by default. I had trouble finding in
documentation way to use Time-Based Tokens so I read [usersfile parsing
code](http://git.savannah.gnu.org/cgit/oath-toolkit.git/tree/liboath/usersfile.c)
which gave me idea how to do this.

For sequential token: <tt>HOTP = HOTP/E = HOTP/E/6</tt>

For 30s Time-Based Token: <tt>HOTP/T30 = HOTP/T30/6</tt>

6 stands for token length. There is also <tt>T60</tt> variant. Available
token lengths are 6, 7 and 7.

In my case I have something like:

{% highlight text %}
HOTP/T30 ivyl - bbad0952f0a72626e216e206d121e314c3ee1700
{% endhighlight %}

Usersfile should not be readable or writable by non-root, so

{% highlight text %}
chmod go-rw /etc/users.oath
{% endhighlight %}

seems like a good idea.

## SSH

Make sure that following options are present in you <tt>sshd_config</tt>:

{% highlight ruby %}
ChallengeResponseAuthentication yes
PasswordAuthentication yes
{% endhighlight %}

Restart <tt>sshd</tt>.

## NTP

Remember to use NTP to sync your clock! I recommend enabling <tt>ntpd</tt> to
start at boot time.

## The Token App

I use [Google
Authenticator](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2).
There are few quirks though. It supports only 30 second time based passwords,
and it expects secret in base32 form (not as hexadecimal representation).

I wrote simple [hex2base32.rb](https://gist.github.com/ivyl/7428982) to aid
me in conversion.

You can find something similar for most mobile platforms, or you just can use
sequential passwords and generate few in advance by <tt>oathtool</tt>.

## Afterthoughts

I recommend digging into code of oath-toolkit. Especially the [PAM
module](http://git.savannah.gnu.org/cgit/oath-toolkit.git/tree/pam_oath/pam_oath.c)
and the [file mentioned
earlier](http://git.savannah.gnu.org/cgit/oath-toolkit.git/tree/liboath/usersfile.c).
PAM simplicity is astonishing. I learned also of <tt>strtok</tt> library
function that comes in handy.

That's it. There's nothing more. You are fine to go and boast off.



