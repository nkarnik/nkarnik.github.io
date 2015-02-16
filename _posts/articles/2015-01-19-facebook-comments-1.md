---
layout: post
title: Where Are My Facebook Commenters? Part 1
excerpt: "If your website or blog uses Facebook comments as its discussion platform, you may be unknowingly sitting on a goldmine of marketing data."
modified: 2015-01-19
categories: articles
tags: [zillabyte,python,facebook,comments,fivethirtyeight,web scraping,web crawling]
comments: true
share: true
search_omit: false
---

If your website or blog uses Facebook comments as its discussion platform, you may be unknowingly sitting on a goldmine of marketing data. In one interpretation, people who take the time to comment on your website (not spam) are some of your most engaged readers. Or enraged readers, depending on how objectionable they find your content :). Given Facebook's aggressive "real name" requirements, having your most engaged readers' valid identities and information can be useful for marketing strategy.

What is one digestible use of this data? A simple exercise is plotting the known locations of your commenters. Not every commenter publicly broadcasts their location, so the input dataset is not perfect, but incomplete is much better than nonexistent. <a href="http://www.fivethirtyeight.com" title="FiveThirtyEight" target="_blank">FiveThirtyEight.com</a> is a leading data journalism website, founded by Nate Silver. In addition to Nate, one of the most prolific authors on the site is the lead writer for FiveThirtyEight's Datalab, Mona Chalabi. Here is a quick comparison of the known locations of commenters from each of their last 50 articles, with comment density by location. 

<iframe width='100%' height='520' frameborder='0' src='http://zillabyte.cartodb.com/viz/e538d1a4-a026-11e4-a8b5-0e0c41326911/embed_map' allowfullscreen webkitallowfullscreen mozallowfullscreen oallowfullscreen msallowfullscreen></iframe>

<iframe width='100%' height='520' frameborder='0' src='http://zillabyte.cartodb.com/viz/831e466c-a02a-11e4-ad2a-0e9d821ea90d/embed_map' allowfullscreen webkitallowfullscreen mozallowfullscreen oallowfullscreen msallowfullscreen></iframe>

All credit for the beautiful and user-friendly mapping software goes to the great folks at <a href="http://www.cartodb.com" title="CartoDB" target="_blank">CartoDB</a>.

These maps are visually interesting and possibly fun for assigning bragging rights, but honestly, the full list of commenters (with their corresponding Facebook UIDS) and comments is much more useful. Facebook's <a href="https://www.facebook.com/ads/audience_manager/" title="Audience Manager" target="_blank">custom audience advertising</a> is very much a walled garden, but it theoretically allows you to do interesting things with highly targeted ads.

If you are a high quality, but niche content producer like <a href="http://www.fivethirtyeight.com" title="FiveThirtyEight" target="_blank">FiveThirtyEight</a>, trying to mass promote your content a la Buzzfeed is probably the equivalent of dumping your advertising budget into the mysterious black holes featured in Interstellar. I haven't tried this personally, so I admit that I'm just speculating.

A much more fruitful strategy of organically growing readership would be to market popular articles directly (using custom audiences) to your existing commenters, who may or may not be currently sharing your content on social channels. There is possibly no group of people more likely to share your content than the very people who are actively contributing to the discussion on your blog (and potentially interacting with the authors themselves).

Crawling Facebook comments is, surprisingly, not extremely straightforward. Because Facebook comments are often not fully accessible unless javascript interactions (such as clicks) are triggered after page load, I built a Zillabyte component that automates most of this process. Once a browser automation or headless browser tool like Selenium or CasperJS is attached to the Facebook Comment virtual frame, the process is pretty standard between sites.

Here is how to use that component in a Zillabyte app:

{% highlight python %}
import zillabyte

app = zillabyte.app(name = "facebook_comments")
stream = app.source_from_csv("urls.csv", headers=["url"])
stream = stream.call_component("facebook_comment_extractor")

sink = stream.sink(name="facebook_comments", columns = [{"full_name":"string"},{"facebook_id":"string"},\
                                              {"page_url":"string"},{"location":"string"}, {"comment":"string"}\
                                              {"author":"string"}, {"page_title":"string"}]

{% endhighlight %}

Example row of the resulting CSV:
Eric Prange,male,"Silver Spring, Maryland",erprange,http://fivethirtyeight.com/datalab/which-state-has-the-worst-drivers/,MONA CHALABI,"Dear Mona, Which State Has The Worst Drivers?",Why are loses per driver only 10-20% of cost of premiums? Is there really that much overhead/profit or am I missing something here?