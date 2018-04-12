---
layout: post
title: "Finnish and Estonian LM training data"
description: ""
category: 
tags: []
---
{% include setup %}

### Conversational Finnish and Estonian data sets

Here are a couple of data sets that I have collected from the web for training
language models:

* [Conversational Finnish (73 million words, 197 MB)](https://dl.dropboxusercontent.com/s/gckrscbkuilfvhm/73M-conversational-finnish.txt.gz)
* [Conversational Estonian (80 million words, 204 MB)](https://dl.dropboxusercontent.com/s/kodlx8maserea3x/80M-conversational-estonian.txt.gz)

The data, originally 2.7 billion Finnish words and 340 million Estonian words,
have been collected by
[crawling conversation sites]({{ site.baseurl }}{% post_url 2017-05-04-crawling-conversation-sites %}).
The text has been normalized and filtered to match transcribed conversations,
duplicate lines have been removed, and the sentences have been shuffled. The
filtering is described in Kurimo et al. (2016),
[Modeling under-resourced languages for speech recognition]({{ site.baseurl }}/publications/lre2016.pdf).
