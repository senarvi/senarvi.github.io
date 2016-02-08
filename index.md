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

## Publications

<ul>
  <li>
    Seppo Enarvi and Mikko Kurimo (2013),
    <a href="publications/iwslt2013.pdf">Studies on Training Text Selection for Conversational Finnish Language Modeling</a>.
    In Proceedings of the 10th International Workshop on Spoken Language Translation (IWSLT 2013).
    (<a href="papers/iwslt2013.pdf">PDF</a>,
    <a href="papers/iwslt2013.bib">BibTex</a>).<br />
    RELATED CODE: <a href="https://github.com/senarvi/senarvi-speech/tree/master/filter-text">filter-text</a>,
    <a href="https://github.com/senarvi/senarvi-speech/tree/master/oov-stats">oov-stats</a>.
  </li>
  <li>
    Seppo Enarvi and Mikko Kurimo (2013),
    <a href="papers/sped2013.pdf">A Novel Discriminative Method for Pruning Pronunciation Dictionary Entries</a>.
    In Proceedings of the 7th International Conference on Speech Technology and Human-Computer Dialogue (SpeD 2013), pages 113&ndash;116.
    (<a href="papers/sped2013.pdf">PDF</a>,
    <a href="papers/sped2013.bib">BibTex</a>).<br />
    NOTE: See erratum at the end of the PDF file.<br />
    RELATED CODE: <a href="https://github.com/senarvi/senarvi-speech/tree/master/filter-dictionary">filter-dictionary</a>.
  </li>
  <li>
    Seppo Enarvi (2012),
    <a href="https://aaltodoc.aalto.fi/handle/123456789/6044">Finnish Language Speech Recognition for Dental Health Care</a>.
    Licentiate thesis, Aalto University School of Science, Espoo.
    (<a href="papers/lic_enarvi_seppo_2012.pdf">PDF</a>,
    <a href="papers/lic_enarvi_seppo_2012.bib">BibTeX</a>)
  </li>
  <li>
    Seppo Enarvi (2006),
    <a href="https://aaltodoc.aalto.fi/handle/123456789/9031">Image-based detection of defective logs</a>.
    Master's thesis, Helsinki University of Technology, Espoo.
    (<a href="papers/master_enarvi_seppo_2006.pdf">PDF</a>,
    <a href="papers/master_enarvi_seppo_2006.bib">BibTeX</a>)
  </li>
</ul>

## Software Projects

<ul>
  <li><a href="https://github.com/senarvi/theanolm/">TheanoLM</a>, recurrent neural network language modeling using Theano</li>
  <li><a href="https://github.com/senarvi/senarvi-freeframe/tree/master/FFGLLightBrush">FFGLLightBrush</a>, FFGL (Resolume) plug-in for light painting</li>
  <li><a href="https://github.com/senarvi/senarvi-unix/tree/master/pm-scripts">pm-scripts</a>, common interface for Red Hat / Debian package managers</li>
</ul>
