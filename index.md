---
layout: page
title: Seppo Enarvi
tagline:
---
{% include JB/setup %}

## Contact Information

Aalto University  
School of Electrical Engineering  
Department of Signal Processing and Acoustics  
Otakaari 5 A  
FI-02150 Helsinki  
Finland

Room: G 336

E-mail: {{ site.author.email }}

Mobile: +358-44-322-5563

GitHub: [{{ site.author.github }}](https://github.com/{{ site.author.github }})

## Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## Software Projects

<ul>
  <li><a href="https://github.com/senarvi/theanolm/">TheanoLM</a>, recurrent neural network language modeling using Theano</li>
  <li><a href="https://github.com/senarvi/senarvi-freeframe/tree/master/FFGLLightBrush">FFGLLightBrush</a>, FFGL (Resolume) plug-in for light painting</li>
  <li><a href="https://github.com/senarvi/senarvi-unix/tree/master/pm-scripts">pm-scripts</a>, common interface for Red Hat / Debian package managers</li>
</ul>
