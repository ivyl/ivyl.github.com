---
layout: page
title: Blag
---
{% include JB/setup %}

## Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>


## My configuration files
[vim](https://github.com/ivyl/vim-config)
[zsh](https://github.com/ivyl/zsh-config)
[i3](https://github.com/ivyl/i3-config)

I found myself often searching for inspiration when working on my dotfiles.
Often I just copy parts of others' files into mine. So here are they - work of
dozens of people (including me!) merged together. As my contribution I share
them with comprehensive READMEs. Enjoy!

## Credits
[Blag's source](https://github.com/ivyl/ivyl.github.com) is available.

I'm using [Jekyll](https://github.com/mojombo/jekyll) and [JekyllBootstrap](http://jekyllbootstrap.com/).
