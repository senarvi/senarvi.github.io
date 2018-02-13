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
Otakaari 3  
FI-02150 Espoo  
Finland

Room: F415b

E-mail: {{ site.author.email }}

Mobile: +358-44-322-5563

## Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## Experience

My CV as a <a href="cv.pdf">PDF</a>

## Code

* [GitHub profile](https://github.com/{{ site.author.github }}/)
* [TheanoLM](https://github.com/senarvi/theanolm), recurrent neural network language modeling using Theano
* [FFGLLightBrush](https://github.com/senarvi/senarvi-freeframe/tree/master/FFGLLightBrush), FFGL (Resolume) plug-in for light painting
* [senarvi-speech](https://github.com/senarvi/senarvi-speech), collection of scripts related to speech recognition and language modeling
* [senarvi-image](https://github.com/senarvi/senarvi-image), C++ classes related to image processing
* [pm-scripts](https://github.com/senarvi/senarvi-unix/tree/master/pm-scripts), common interface for Red Hat / Debian package managers

## Publications

* Peter Smit, Siva Reddy Gangireddy, Seppo Enarvi, Sami Virpioja, and Mikko Kurimo (2017),
  [Character-Based Units for Unlimited Vocabulary Continuous Speech Recognition](http://dx.doi.org/10.1109/ASRU.2017.8268929).
  In Proceedings of the 2017 IEEE Automatic Speech Recognition and Understanding Workshop (ASRU).
  ([PDF]({{ site.baseurl }}/publications/asru2017.pdf),
  [BibTex]({{ site.baseurl }}/publications/asru2017.bib)).
* Peter Smit, Siva Reddy Gangireddy, Seppo Enarvi, Sami Virpioja, and Mikko Kurimo (2017),
  [Aalto System for the 2017 Arabic Multi-Genre Broadcast Challenge](http://dx.doi.org/10.1109/ASRU.2017.8268955).
  In Proceedings of the 2017 IEEE Automatic Speech Recognition and Understanding Workshop (ASRU).
  ([PDF]({{ site.baseurl }}/publications/asru2017mgb.pdf),
  [BibTex]({{ site.baseurl }}/publications/asru2017mgb.bib)).
* Seppo Enarvi, Peter Smit, Sami Virpioja, and Mikko Kurimo (2017),
  [Automatic Speech Recognition with Very Large Conversational Finnish and Estonian Vocabularies](http://dx.doi.org/10.1109/TASLP.2017.2743344).
  In IEEE/ACM Transactions on Audio, Speech, and Language Processing.
  ([PDF]({{ site.baseurl }}/publications/taslp2017.pdf),
  [BibTex]({{ site.baseurl }}/publications/taslp2017.bib)).  
  NOTE: See erratum at the end of the PDF file.
* Seppo Enarvi and Mikko Kurimo (2016),
  [TheanoLM — An Extensible Toolkit for Neural Network Language Modeling](http://www.isca-speech.org/archive/Interspeech_2016/abstracts/0618.html).
  In Proceedings of the 17th Annual Conference of the International Speech Communication Association (INTERSPEECH), pp. 3052–3056.
  ([PDF]({{ site.baseurl }}/publications/interspeech2016.pdf),
  [BibTex]({{ site.baseurl }}/publications/interspeech2016.bib)).  
  RELATED CODE: [TheanoLM](https://github.com/senarvi/theanolm).
* Mikko Kurimo, Seppo Enarvi, Ottokar Tilk, Matti Varjokallio, André
  Mansikkaniemi, and Tanel Alumäe (2016),
  [Modeling under-resourced languages for speech recognition](http://dx.doi.org/10.1007/s10579-016-9336-9).
  Language Resources and Evaluation.
  ([PDF]({{ site.baseurl }}/publications/lre2016.pdf),
  [BibTex]({{ site.baseurl }}/publications/lre2016.bib)).  
  NOTE: See erratum at the end of the PDF file.  
  RELATED CODE: [filter-text](https://github.com/senarvi/senarvi-speech/tree/master/filter-text),
  [filter-dictionary](https://github.com/senarvi/senarvi-speech/tree/master/filter-dictionary).
* Seppo Enarvi and Mikko Kurimo (2013),
  [Studies on Training Text Selection for Conversational Finnish Language Modeling](http://workshop2013.iwslt.org/downloads/Studies_on_Training_Text_Selection_for_Conversational_Finnish_Language_Modeling.pdf).
  In Proceedings of the 10th International Workshop on Spoken Language Translation (IWSLT), pp. 256–263.
  ([PDF]({{ site.baseurl }}/publications/iwslt2013.pdf),
  [BibTex]({{ site.baseurl }}/publications/iwslt2013.bib)).  
  RELATED CODE: [filter-text](https://github.com/senarvi/senarvi-speech/tree/master/filter-text),
  [oov-stats](https://github.com/senarvi/senarvi-speech/tree/master/oov-stats).
* Seppo Enarvi and Mikko Kurimo (2013),
  [A Novel Discriminative Method for Pruning Pronunciation Dictionary Entries](http://dx.doi.org/10.1109/SpeD.2013.6682659).
  In Proceedings of the 7th Conference on Speech Technology and Human-Computer Dialogue (SpeD), pp. 113–116.
  ([PDF]({{ site.baseurl }}/publications/sped2013.pdf),
  [BibTex]({{ site.baseurl }}/publications/sped2013.bib)).  
  NOTE: See erratum at the end of the PDF file.  
  RELATED CODE: [filter-dictionary](https://github.com/senarvi/senarvi-speech/tree/master/filter-dictionary).
* Seppo Enarvi (2012),
  [Finnish Language Speech Recognition for Dental Health Care](http://urn.fi/URN:NBN:fi:aalto-201210203324).
  Licentiate thesis, Aalto University School of Science, Espoo.
  ([PDF]({{ site.baseurl }}/publications/lic_enarvi_seppo_2012.pdf),
  [BibTex]({{ site.baseurl }}/publications/lic_enarvi_seppo_2012.bib))
* Seppo Enarvi (2006),
  [Image-based detection of defective logs](http://urn.fi/URN:NBN:fi:aalto-201304271936).
  Master's thesis, Helsinki University of Technology, Espoo.
  ([PDF]({{ site.baseurl }}/publications/master_enarvi_seppo_2006.pdf),
  [BibTex]({{ site.baseurl }}/publications/master_enarvi_seppo_2006.bib))
