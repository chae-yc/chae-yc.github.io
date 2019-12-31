---
permalink: /about/
title: "About"
excerpt: "It's me"
layouts_gallery:
  - url: /assets/images/yunchang.jpeg
    image_path: /assets/images/yunchang.jpeg
    alt: "Yunchang Chae"

last_modified_at: 2019-12-31T20:27:00+09:00
toc: true
---

<!--
This is my personal blog. It currently has {{ site.posts | size }} posts in {{ site.categories | size }} categories which combinedly have {{ total_words }} words, which will take an average reader ({{ site.wpm }} WPM) approximately <span class="time">{{ total_readtime }}</span> minutes to read. {% if featuredcount != 0 %}There are <a href="{{ site.url }}/featured">{{ featuredcount }} featured posts</a>, you should definitely check those out.{% endif %} The most recent post is {% for post in site.posts limit:1 %}{% if post.description %}<a href="{{ site.url }}{{ post.url }}" title="{{ post.description }}">"{{ post.title }}"</a>{% else %}<a href="{{ site.url }}{{ post.url }}" title="{{ post.description }}" title="Read more about {{ post.title }}">"{{ post.title }}"</a>{% endif %}{% endfor %} which was published on {% for post in site.posts limit:1 %}{% assign modifiedtime = post.modified | date: "%Y%m%d" %}{% assign posttime = post.date | date: "%Y%m%d" %}<time datetime="{{ post.date | date_to_xmlschema }}" class="post-time">{{ post.date | date: "%d %b %Y" }}</time>{% if post.modified %}{% if modifiedtime != posttime %} and last modified on <time datetime="{{ post.modified | date: "%Y-%m-%d" }}" itemprop="dateModified">{{ post.modified | date: "%d %b %Y" }}</time>{% endif %}{% endif %}{% endfor %}. The last commit was on {{ site.time | date: "%A, %d %b %Y" }} at {{ site.time | date: "%I:%M %p" }} [UTC](http://en.wikipedia.org/wiki/Coordinated_Universal_Time "Temps Universel Coordonné").
-->

## Resume
![Resume]("Idon'tKnowWhereItIs"){: .btn .btn--success .btn--large}

{% include gallery id="layouts_gallery" caption="Examples of included layouts `splash`, `single`, and `archive`." %}

## INTRODUCTION

## EXPERIENCE
###   - *Team project*
<sub>from - to, [Github](link), [Report](link)</sub>
- todo

## EDUCATION
### 학사 - *경북대학교 컴퓨터학부*
졸업예정 <sub>2014.03.02 - 2020.02.21</sub>

## Skills

### Language
Python, JavaScript, C

### Framework
Android, [TensorFlow](https://github.com/newhiwoong/TensorFlow)


## Contact
✉️ [yunchang.chae@gmail.com]()  
🌐 [https://github.com/chae-yc](https://github.com/chae-yc)
