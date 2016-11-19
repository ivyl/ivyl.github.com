---
layout: post
title: "SSH Time-Based One-Time Passwords"
tags: [unix, security]
---
{% include JB/setup %}


*"I do not alway log in remotely through unsafe machines, but when I do, I do
it as securely as possible."*


## Intro

In order to keep my systems safe, even in case the passwords had been
overheard/peeped/logged, I use 2-step authentication.

You should too.

I prefer [Time-Based One-Time Passwords][totp_rfc] instead of the one that
[increments the counter][hotp_rfc]. There are quite a few implementations of
this idea available. Reading through I have chosen to go with [OATH][oath]
which is industry-wide effort to provide such solutions basing on open
standards.

I alway carry my phone with me, so it was natural candidate to serves me as
token generator. I have installed and FOSS app, [FreeOTP][freeotp], that
generates the tokens.

Also, since it's all time based, I made sure that clocks on the devices are
synced up - I am using carrier synchronization on my phone and NTP clients are
running on the servers.


## OATH

You should install [oath-toolkit][oath_toolkit], which is provides some
components and libraries we are going to use, including PAM module to handle
the authentication.

```bash
portmaster security/oath-toolkit
dnf install oath-toolkit
pacman -S oath-toolkit
```

Or however it's done in your *religion*.


## PAM

In `/etc/pam.d/` you should find files corresponding to logging in method, such
as `sshd`, `su`, etc.

I added auth module as *requisite* (see `man pam.conf` for details) on
top of ssh one.

```
auth requisite /usr/local/lib/security/pam_oath.so usersfile=/etc/users.oath
```

**On FreeBSD** I had to specify full path to the module.

**On most Linux** distributions, if oath-toolkit is installed through package
manager using `pam_oath.so` instead of full path should be sufficient.

You can go with *required* instead of *requisite* if you don't want to reveal
whether the OTP was correct straight away.

From now on you will be asked for OTP first, and then for you'll have to
provide your regular password.


## The Userfile

The **usersfile** I pointed OATH module to is `/etc/users.oath`.

**Fedora caveat:** use `/etc/liboath/` directory, due to selinux permissions.

Example provided in documentation is very brief:

```
HOTP root - 00
```

Too brief, I would say. It took me some research (actually reading through
source code) to make sense of the syntax.

White spaces are threated as delimiter, and fields are:

* **first** is auth method,
* **second** is user name
* **third** would be used if we would like to have additional password
  provided along with the token (since in this case PAM will ask us for
  the regular password just after token, we are okay with none using value `-`)
* **fourth field** is the secret used to generate tokens, in hexadecimal format.
* **fields after that** would contain last password used (so it can't be
  reused straight away) as well as the counter regular HOTP


I recommend using SHA1 of a random data as a secret:

```bash
head -c 2048 /dev/urandom | openssl sha1
```

HOTP is sequentional 6 digit key by default. I had trouble finding in
documentation way to use Time-Based Tokens so I read [usersfile parsing
code][oath_userfile_source] which gave me idea how to do this.

For sequential token: `HOTP = HOTP/E = HOTP/E/6`

For 30s Time-Based Token: `HOTP/T30 = HOTP/T30/6`

6 stands for token length. There is also `T60` variant. Valid
token lengths are 6, 7 and 8.

In my case I have something in the lines of:

```
HOTP/T30 ivyl - bbad0952f0a72626e216e206d121e314c3ee1700
```

Usersfile should not be readable or writable by non-root, so just do

```
chmod go-rw /etc/users.oath
```

## SSH

Make sure that following options are present in you `sshd_config`:

```ruby
ChallengeResponseAuthentication yes
PasswordAuthentication yes
```

Otherwise the OTP challenge won't appear while logging in.


## The Token App

I use [FreeOTP][freeotp]. The secret should be base32 encoded (instead of
simple hexadecimal dump)... You can use `oathtool` provided with the
oath-toolkit to do the conversion.

```bash
oathtool -v $(read foo; echo $foo)
```

And paste the hexadecimal secret.


## Afterthoughts

I recommend digging into code of oath-toolkit. Especially the [PAM
module][oath_pam_source]. PAM simplicity is astonishing.

Also read through [HOTP][hotp_rfc] and [TOTP][totp_rfc] RFCs.


## UPDATES:

* <tt>20161119</tt> - general overhaul


[hex2base32]: https://gist.github.com/ivyl/7428982
[oath]: http://www.nongnu.org/oath-toolkit/
[oath_toolkit]: http://www.nongnu.org/oath-toolkit/
[oath_userfile_source]: http://git.savannah.gnu.org/cgit/oath-toolkit.git/tree/liboath/usersfile.c
[oath_pam_source]: http://git.savannah.gnu.org/cgit/oath-toolkit.git/tree/pam_oath/pam_oath.c
[totp_rfc]: https://tools.ietf.org/html/rfc6238
[hotp_rfc]: https://tools.ietf.org/html/rfc4226
[freeotp]: https://freeotp.github.io/

