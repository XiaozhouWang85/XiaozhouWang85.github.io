---
layout: default
---
<h3>
<a href="/About/" style="color:#324152">About Me</a>
</h3>
I’m a senior analytics manager and aspiring data scientist working in the financial services industry. I have 10 years experience in traditional analytics (SAS, SQL) including managing small teams of analysts. I've spent the last 3 years making the transition to using Python for data science on a personal level and am now helping my workplace make a similar transition.

Learning to use Python has been a truly empowering process and what's been amazing is the pace at which new data science packages are being released. I'm starting this blog in order to share my experiences with others who are also learning Python for data science.

While my blog will focus on purely technical topics, please take a look at the "About Me" section and my CV to find out about my non-technical skills and interests outside of work.

![Profile]({{site.url}}/assets/profile.JPG){:class="img-responsive" width="100%"}

<h3>
<a href="/blog/" style="color:#324152">Recent Blog Posts</a>
</h3>

{% for post in site.posts limit:4 %}
[{{ post.title }}]({{ post.url }}) {{ post.date | date: '%B %d, %Y' }}

{{ post.excerpt | remove: '<p>' | remove: '</p>' }}
{% endfor %}

  
  
