---
layout: page
title: Hello
tagline:
---
{% include JB/setup %}

I don't feel like writing a presentation right now. Maybe later.


## Repos

<ul>
    <li><a href="https://www.github.com/alols/xcape">xcape</a></li>
    <li><a href="https://www.github.com/alols/dotfiles">dotfiles</a></li>
</ul>

## Blog posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

