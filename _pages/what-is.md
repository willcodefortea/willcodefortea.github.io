---
permalink: /what-is/
title: What Is...?
layout: archive
---

I spend a lot of time _talking_. Probably more than I should and definitely more than people enjoy! Along the way however, I often get asked common programming questions and while most of the time I have a good answer to hand, it can be useful to have a record of those as well as a good opportunity to make sure what I'm talking about is correct.. 👀

{% for post in site.what_is %}
  {% include archive-single.html %}
{% endfor %}
