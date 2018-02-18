---
layout: default
---
<h3>
<a href="/About/" style="color:#324152">About Me</a>
</h3>
Learn more about my present past life and background.

<h3>
<a href="https://github.com/XiaozhouWang85" style="color:#324152">Projects</a>
</h3>
To Come

<h3>
<a href="/blog/" style="color:#324152">Recent Blog Posts</a>
</h3>

{% for post in site.posts limit:4 %}
[{{ post.title }}]({{ post.url }}) {{ post.date | date: '%B %d, %Y' }}

{{ post.excerpt | remove: '<p>' | remove: '</p>' }}
{% endfor %}

  
  
