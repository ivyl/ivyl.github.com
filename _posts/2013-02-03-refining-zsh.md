---
layout: post
title: "Refining ZSH"
tags: [shell, zsh, tools]
---
{% include JB/setup %}


# Table of Contents

* This will become a table of contents (this text will be scraped).
{:toc}

## Foreword

Shell is something you use a lot. Launching commands, navigating around, and
even programming. It is not only tool of trade for programmers and system
admins, it essential tool for everyday usage that can be refined and tailored
to suit specific needs. It takes some time to get your head around it
and do all the customization, but it's worth every second spent on it.

Zsh is getting more and more popular nowadays. It's really great as
interactive shell having lots of features and being easily configurable and
well documented. I made my switch from Bash two years ago. I never looked
back.

I find frameworks such as
[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh) and
[grml](http://grml.org/zsh/) (which I seen referred as The Missing Zsh
Default Configuration) bloated. I hand-tailored [my
Zsh](https://github.com/ivyl/zsh-config) to be no more and no
less what I expect it to be. I want to share few things I stumbled upon on the
internet/comprehensive Zsh man pages or set up myself. You won't find common/obvious stuff
like setting completion/prompt right though. I want to focus on bit more
obscure features.

I hope you will find something interesting here, that will make you work more
fun.

**NOTE:** Color definitions seen below are from [this
file](https://github.com/ivyl/zsh-config/blob/master/colors.zsh).

**NOTE2:** setting Zsh options is done by `setopt OPTION`.

**NOTE3:** I'm aware that some of things mentioned here works in bash as well.

## Tricks worth knowing

### Launching command omitting aliased stuff

Just prepend it with backslash:

{% highlight sh %}
alias ls='ls -lah --color'
\ls
{% endhighlight %}

Other posssible way to do that is use `=` operator which returns location of
executable file (similar to "which" command):

{% highlight sh %}
echo =ls #=> /usr/bin/ls
=ls
{% endhighlight %}

I recommend you to play with = for a while and get use to it.

{% highlight sh %}
ls -l `which ls`
ls -l =python
echo =ls
which ls
{% endhighlight %}

See **EQUALS** option (on by default).

### Multiple redirections

{% highlight sh %}
cat < file1 < file2 > file3
{% endhighlight %}

Which makes stdin/err redirections much easier:

{% highlight sh %}
\time ls > file.stdin 2> file.stderr
{% endhighlight %}


### Last command

In shell `!!` is substituted with last command. Simplest use case:

{% highlight sh %}
chown ivyl file #=> permission denided
sudo !! #=> sudo chown ivyl file
{% endhighlight %}

When option **HIST_VERIFY** is set then you are able to edit full command after
substitution.

There are fancier methods of inserting last command in-place. My favorite is one
doing it along with substitution:

{% highlight sh %}
cat fooberbaz #=> no such file
^ber^bar #=> cat foobarbaz
{% endhighlight %}

### Clobber - how to not override existing file

Does overwriting exiting file with stream redirection `>` ever happened to
you? Was it painful? Worry not. Just:

{% highlight sh %}
setopt NOCLOBBER
{% endhighlight %}

Which results in:

{% highlight sh %}
cat > foo #=> Zsh: file exists: foo
{% endhighlight %}

To override file use `>|` instead of `>`.

{% highlight sh %}
cat >| foo #=> OK
{% endhighlight %}

You can set `HIST_ALLOW_CLOBBER` to have `|` added by default in history
entries, so that successful retry is just two keystrokes away.

### Bashquese Ctrl-W

In Zsh ^W removes words delimited by whitespace. We are working in shell here
though, this should be more fine-grained. I like how it behave in bash -
slashes, dots and few other things are treated as delimiters too. You can achieve
this behaviour in Zsh by simply:

{% highlight sh %}
autoload -U select-word-style
select-word-style bash
{% endhighlight %}

### Easy variable editing

The way to do it is to use `vared`. Type `vared PROMPT` and Zsh Line Editor
will be launched enabling you to edit it. It's great when combined with Vi
Mode.

You can imagine it is useful for messing with `$PATH`, experimenting with `$PROMPT`.

### In-place expanding

When entering some expression that will be expanded with Zsh
(substituted/interpolated) you can hit TAB key just after it to do it in
place and see result right in command line. You might stumbled upon this
feature by accident. Few examples to try:

{% highlight sh %}
ls *<TAB>
ls -l `which grep`<TAB>
ls -l =grep<TAB>
{% endhighlight %}

I find it useful to peek on what is done before command will be executed.
Sometimes I find removing one match from list easier than constructing fancy
pattern.

### Expansion aka substitution aka globbing

I guess that you are familiar with wildcards such as `*` and `**` which
matches directories recursively. For more specific searches you had to use
`find`. It's ok for scripts and very detailed searches but in ZSH most
use cases can 

Ever annoyed by directory errors  when greping with `**/*`? Append `(.)` to
limit match only to files.

{% highlight sh %}
grep pattern **/*(.)
{% endhighlight %}

`(/)` does the same with directories:
{% highlight sh %}
for i in **/*(/); echo $i
{% endhighlight %}

This is quite well known but it's worth mentioning:
{% highlight sh %}
vimdiff file{,.bak}
{% endhighlight %}

`file.{a,b,cd}` will be expanded to `file.a file.b file.cd`. The trick above
ise to use empty mach to edit regular file as well as it .bak version. Menu
completion works very well with this.

There is a lot of more to it. I recomend you watching [great
video](http://openclassroom.stanford.edu/MainFolder/VideoPage.php?course=PracticalUnix&video=zsh-globbing)
on this topic as well as reding [one of many
cheatsheats](https://github.com/gaving/wisdom/blob/master/data/zsh-glob).

### Mass renaming via zmv

Zmv is great tool for mass renameing. Since I found it I don't even have any file manager installed.

{% highlight sh %}
autoload -Uz zmv
{% endhighlight %}

As it help states:

{% highlight text %}
Usage:
  zmv [OPTIONS] oldpattern newpattern
where oldpattern contains parenthesis surrounding patterns which will
be replaced in turn by $1, $2, ... in newpattern.  For example,
  zmv '(*).lis' '$1.txt'
renames 'foo.lis' to 'foo.txt', 'my.old.stuff.lis' to 'my.old.stuff.txt',
and so on.  Something simpler (for basic commands) is the -W option:
  zmv -W '*.lis' '*.txt'
This does the same thing as the first command, but with automatic conversion
of the wildcards into the appropriate syntax.  If you combine this with
noglob, you don't even need to quote the arguments.  For example,
  alias mmv='noglob zmv -W'
  mmv *.c.orig orig/*.c
{% endhighlight %}

### History options

Few useful history options:

{% highlight sh %}
setopt EXTENDED_HISTORY
setopt INC_APPEND_HISTORY
setopt HIST_IGNORE_ALL_DUPS
setopt HIST_IGNORE_SPACE
setopt HIST_REDUCE_BLANKS
setopt HIST_VERIFY
{% endhighlight %}

As an excerise I recommend you searching it in Zsh man pagaes. Use `man
zshoptions` or `man zshall`.

## Bindings

### Edit current line in text editor

Bash has this feature out of the box. In zsh we need to setup it manualy:

{% highlight sh %}
autoload -U edit-command-line
zle -N edit-command-line
bindkey '^f' edit-command-line
{% endhighlight %}

I have choses Ctrl-F bindign to keep it similar to Vim. edit-command-line
uses $EDITOR variable.

### Vi mode and Escape time out

Vi mode makes you shell modal. It will put you in insert mode by default
(which is saner option) and allow you to use most of the Vim movements from
command/normal mode after pressing enter. You search history by `/` and `?`.

To enable it:

{% highlight sh %}
bindkey -v
{% endhighlight %}

It will take some time to get used to. For Vim users this makes much more
homogeneous environment.

I recommend setting KEYTIMEOUT to lower value. It's time Zsh waits for key
escape key sequence. Since I don't use those this latency annoys me when it
comes to search (/). I use ^\[ instead of escape and I type so fast that it
often have undesired effects triggering some strange stuff. Default is 40 (in
hundredths of second) witch is bit high for me. 

{% highlight sh %}
export KEYTIMEOUT=1
{% endhighlight %}

### Vi mode enhancements

{% highlight sh %}
# ctrl-p ctrl-n history navigation
bindkey '^P' up-history
bindkey '^N' down-history

# backspace and ^h working even after returning from command mode
bindkey '^?' backward-delete-char
bindkey '^h' backward-delete-char

# ctrl-w removed word backwards
bindkey '^w' backward-kill-word

# ctrl-r starts searching history backward
bindkey '^r' history-incremental-search-backward
{% endhighlight %}

Other big enhancment is prompt that tells us which mode we are in by cursor
colour. Read on to get to that.


## Prompt

### VCS (Git, Subversion, Mercurial) prompt

{% highlight sh %}
# load module
autoload -Uz vcs_info

# set style for vcs info
zstyle ':vcs_info:*' stagedstr "${fg_blue}?"
zstyle ':vcs_info:*' unstagedstr "${fg_brown}?"
zstyle ':vcs_info:*' check-for-changes true
zstyle ':vcs_info:(sv[nk]|bzr):*' branchformat '%b%F{1}:%F{11}%r'
zstyle ':vcs_info:*' enable git svn

# executed before each command to change style including red ! for unstaged
# changed
precmd () {
    if [[ -z $(git ls-files --other --exclude-standard 2> /dev/null) ]] {
        zstyle ':vcs_info:*' formats "${at_normal} ${fg_dgray}%b%c%u${at_normal}"
    } else {
        zstyle ':vcs_info:*' formats "${at_normal} ${fg_dgray}%b%c%u${fg_red}!${at_normal}"
    }
 
    vcs_info
}
 
# enable later substitution in prompt
setopt PROMPT_SUBST

PROMPT="%c \${vcs_info_msg_0_}"
{% endhighlight %}

**Note** how dollar sign is escaped. If you forget it the expansion of
variable will happen once. You could use single quotes as well.

Setting above will result in having branch and status information (indicated
by colorful punctuation marks) in your prompt.

### Exit Status

{% highlight sh %}
PROMPT="%(?/${at_normal}/${fg_red})%%${at_normal}"
{% endhighlight %}

The above will set your prompt to coloured percent sign (red/normal) depending on last
command exit status.

### Vi mode status indicator

It's hard  to tell in which mode Zsh is. I have seen some tricks including
PS2, but I liked cursor colour the most.

My version works well in tmux and doesn't mess Linux Console. It also fixed
issue with cursor staying red when launched commend from normal mode.

{% highlight sh %}
# urxvt (and family) accepts even #RRGGBB
INSERT_PROMPT="gray"
COMMAND_PROMPT="red"

# helper for setting color including all kinds of terminals
set_prompt_color() {
    if [[ $TERM = "linux" ]]; then
       # nothing
    elif [[ $TMUX != '' ]]; then
        printf '\033Ptmux;\033\033]12;%b\007\033\\' "$1"
    else
        echo -ne "\033]12;$1\007"
    fi
}

# change cursor color basing on vi mode
zle-keymap-select () {
    if [ $KEYMAP = vicmd ]; then
        set_prompt_color $COMMAND_PROMPT
    else
        set_prompt_color $INSERT_PROMPT
    fi
}

zle-line-finish() {
    set_prompt_color $INSERT_PROMPT
}

zle-line-init () {
    zle -K viins
    set_prompt_color $INSERT_PROMPT
}

zle -N zle-keymap-select
zle -N zle-line-init
zle -N zle-line-finish
{% endhighlight %}

It's possible thanks to powerful [Zsh Line
Editor](http://zsh.sourceforge.net/Doc/Release/Zsh-Line-Editor.html).


## Afterword

Post your configuration on Github/elsewhere - you will be just one clone away from your environment and you will share with others.

This is only tip of an iceberg. Check other people's dotfiles on github (Google
-> site:github.com zsh), read man, search zshall man page for interesting
keywords. Keep on tailoring.

