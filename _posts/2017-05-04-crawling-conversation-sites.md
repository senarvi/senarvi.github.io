---
layout: post
title: "Crawling conversation sites"
description: "How to use Scrapy to obtain vast amounts of data for language modeling"
category: 
tags: []
---
{% include JB/setup %}

### Scraping static web pages

In the past, a common way to obtain
[language model training data from the web](https://ssli.ee.washington.edu/tial/projects/ears/WebData/web_data_collection.html)
has involved generating Google queries using in-domain data. When trying to
collect Finnish conversations, I found this method very inefficient. It might be
that something has changed in how Google handles queries, or less Finnish data
is available, or that my set of in-domain n-grams was too small. In any case, I
found crawling large conversation sites using Python and
[Scrapy](https://scrapy.org/) to be way more efficient.

Crawling a site that uses static HTML pages using Scrapy is straightforward.
Every site structure is a bit different, though, so you'll have to customize the
spider for each site. You need to specify the rules for following links to
different conversations, and implement a function that parses the messages
from a conversation page. Creating a new project has been made easy. After
[installing Scrapy](https://doc.scrapy.org/en/latest/intro/install.html),
create the directory structure using:

```bash
scrapy startproject mysite
```

The actual spider class you'll have to implement in
`mysite/spiders/mysite_spider.py`. This is how the file usually starts:

```python
from scrapy.contrib.spiders import CrawlSpider, Rule
from scrapy.contrib.linkextractors.sgml import SgmlLinkExtractor
from scrapy.selector import HtmlXPathSelector
from mysite.items import MessageItem

class MySpider(CrawlSpider):
    name = "mysite"
    allowed_domains = ["mysite.com"]
    start_urls = ["http://mysite.com/viewforum.php"]
    extractor = SgmlLinkExtractor(allow=('viewforum\.php\?f='))
    rules = (
        Rule(extractor, callback='parse_item', follow=True),
    )

    def parse_item(self, response):
        # Parse messages from the response.
```

A CrawlSpider automates the process of following links on the loaded web pages.
`start_urls` defines one or more top-level URLs, where to start crawling. You
might want to give the start page or list all the individual conversation areas.
`allowed_domains` limits the spider to links that point to this domain. You
probably want to stay inside the same domain. `extractor` is an object that
extracts the links from a web page that the spider will follow. You should
provide a regular expression in the `allow` parameter that matches only to the
pages that display conversations.

Implementation of the `parse_item` function depends on the HTML structure of the
site. It should iterate through all the messages in the HTML code, and yield a
new item on each message. Iterating through the messages is facilitated by
[XPath selectors](https://doc.scrapy.org/en/0.10.3/topics/selectors.html), a
mechanism for identifying HTML elements. At its simplest, the function could
look something like this:

```python
    hxs = HtmlXPathSelector(response)
    for message in hxs.select('div[@class="message"]'):
        ids = message.select('@id').extract()
        if len(ids) != 1:
            continue
        id = ids[0]

        text = '\n'.join(message.select('text()').extract())
        if not text:
            continue

        item = MessageItem()
        item['id'] = id
        item['text'] = text
        yield item
```

As you can see, an item contains both an identifier and the message text. The
identifier will be used to make sure that the same message won't be saved more
than once. The XPath `div[@class="message"]` is used to select all `<div>`
elements from the document with class name "message". Proper insructions to
using XPath selectors are out of the scope of this post, but there are a few
things that can be noted from the code. The `select()` method always returns a
list, because there can be multiple elements that match the search. An
attractive feature is that the returned objects are selectors themselves that
can be used to select nested objects. In this case the `<div>` selector is used
to select its id attribute and the text inside the element. The `extract()`
method is used to convert a selector to a text string.
