---
# You don't need to edit this file, it's empty on purpose.
# Edit theme's home layout instead if you wanna make some changes
# See: https://jekyllrb.com/docs/themes/#overriding-theme-defaults
layout: default
---
<h2>About me</h2>
I am a full stack developer with 8 years commercial experience across multiple languages including PHP, C# and JavaScript.

<h2>Blog posts</h2>

{% for post in site.posts %}
  <h3><a href="{{ post.url }}">{{ post.title }}</a><br />
  <small>{{ post.date | date: "%-d %b %Y at %H:%M" }}</small></h3>
  {{ post.blurb }}
{% endfor %}
