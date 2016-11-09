---
layout: page
title: INDEX
---
{% include JB/setup %}

## Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span class="index_post_date">{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## ReSources

**Dotfiles:**
[zsh](https://github.com/ivyl/zsh-config) |
[neovim](https://github.com/ivyl/vim-config) |
[i3](https://github.com/ivyl/i3-config)

**Blag's source**: <https://github.com/ivyl/ivyl.github.com>


