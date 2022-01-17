---
permalink: /what-is/
title: What Is...?
layout: archive
---

I spend a lot of time _talking_. Probably more than I should and definitely more than people enjoy! Along the way however, I often get asked common programming questions, and while most of the time I have a good answer to hand it can be useful to have a reference.

(As well as a good opportunity to make sure what I'm talking about is reasonably correct.. 👀)

> This project started in Dec 2021, let's see if I can add one a week for a year. If there's a topic you think I should cover then feel free to ping me. (I doubt there's anyone here I don't already know so you should be okay! 🙃)

{% assign what_is_posts = site.what_is | sort: "date" | reverse %}
{% for post in what_is_posts %}
  {% include archive-single.html %}
{% endfor %}
