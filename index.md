---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---
<p>
  {% for post in site.posts %}
    <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
    <h5>Posted {{ post.date | date: "%-d %B %Y" }}</h5>
    <p>{{ post.excerpt }}</p>
    <hr>
  {% endfor %}
</p>