---
---

{% for post in site.posts %}

[{{ post.title }}]({{ post.url }})

{% endfor %}

#### Categories

{% for category in site.categories %}

- {{ category[0] }}
  {% endfor %}
