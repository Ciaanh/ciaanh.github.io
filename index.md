---
layout: default
---


{% for post in site.posts %}
<section>
  <h2>
    <a href='{{ site.url }}{{ post.url }}' title='{{ post.title }}'>
      {{ post.title }}
    </a>
  </h2>
  
  <p>{{ post.excerpt | strip_html | truncate: 250 }}</p>
</section>       
{% endfor %}