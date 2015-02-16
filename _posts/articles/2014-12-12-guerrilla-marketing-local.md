---
layout: post
title: Guerrilla Marketing - Target Local Ads Near Your Competitors
excerpt: "Using store locators to manually scrape the locations of all of your competitors' stores is a time consuming and often incomplete process."
modified: 2015-01-19
categories: articles
tags: [zillabyte,python,facebook,comments,fivethirtyeight,web scraping,web crawling]
comments: true
share: true
search_omit: false
---

For e-commerce companies, cutthroat competition is an unavoidable fact of life. What if there were a way to leverage insights from competitors' behavior? E-commerce companies' main competitors are brick and mortar retail stores which sell an identical or substitute product.  A company like Warby Parker may want to know where all offline eyewear retailers' storefronts are located. 
(<em>CartoDB map of most of the Visionworks and Lenscrafters in CA):</em>

<iframe width='100%' height='520' frameborder='0' src='http://zillabyte.cartodb.com/viz/b5363fc8-825d-11e4-9e60-0e9d821ea90d/embed_map' allowfullscreen webkitallowfullscreen mozallowfullscreen oallowfullscreen msallowfullscreen></iframe>

For brick and mortar stores, nothing is more telling than where they choose to put up their storefronts. In the same way that a surf shop is less likely to open up in Nebraska, large retail brands spend a lot of time and money optimizing their store locations (using factors including population density and target demographics, limited by the cost/availability of real estate). At the least, targeting local ads near the locations of offline competitors will reach some significant percentage of your target demographic.

For your e-commerce business, you probably have a list of brands that compete in your vertical. Using store locators to manually scrape the locations of all of your competitors' stores is a time consuming and often incomplete process.

<h1>How Zillabyte Solved this Problem</h1>

Zillabyte Components are reusable pieces of data science. For common but difficult data related tasks, we have built an assortment of highly targeted and useful components. Our most popular component to date is the "domain_crawl" component that performs a deep crawl on a targeted domain.

For the common problem of retreiving business locations, I built a component called "business_locator" which takes an input list of businesses (e.g. Target, H&M, Lenscrafter, etc.) and outputs a CSV of all of the global physical locations of each input business. The <a href="https://gist.github.com/nkarnik/a2cd09e2f063d3bbd817" title="Input CSV" target="_blank">input CSV</a> is simply of the form:


{% highlight python %}
J. Crew
Men's Warehouse  
Zara  
Macy's  
Brooks Brothers
{% endhighlight %}

Under the hood, the component processes and cross-references a variety of distributed API calls to 3rd party services such as Google Places and Yelp. 

<h1>How to Use the Component: </h1>

To demonstrate the power of this component, I will be making a call to the "business_locator" component within a complete Zillabyte app. 

This app sources from the input CSV, which is a list of businesses to find. This is the entire python app:

{% highlight python %}
import zillabyte

# Register the app, source from the input CSV, and call the business_locator component. 
app = zillabyte.app(name = "zilla_commerce")
stream = app.source_from_csv("businesses.csv", headers=["business"])
stream = stream.call_component("business_locator")

#Finally, sink the data to your output CSV
sink = stream.sink(name="business_locations", columns = [{"business":"string"},{"latitude":"float"},\
                                              {"longitude":"float"},{"full_address":"string"},\
                                              {"zip_code":"integer"}]

{% endhighlight %}

It's as simple as it looks. Populate your businesses.csv file with the businesses you're interested in, and call the "business_locator" component. Get your data by running "zillabyte pull:data business_locations output" from the command line. Sample of the output CSV from the map of eyewear stores near the top of the post:

{% highlight python %}
lenscrafter,37.787946,-122.403076,685 Market St,CA,94105
lenscrafter,33.721162,-117.792653,13662 Jamboree Rd,CA,92602
lenscrafter,37.292652,-121.989343,1620 Saratoga Ave,CA,95129
lenscrafter,37.836586,-122.293834,5707 Christie Ave,CA,94608
lenscrafter,34.076249,-118.375763,8471 Beverly Blvd,CA,90048
{% endhighlight %}

<h1>Why Bother? Is this actually useful?</h1>

Yes, absolutely! Even though e-commerce happens in nebulous cyberspace, your customers are always physically somewhere in "meatspace." Brick and mortar stores are limited by physical location, so they use immense resources to ensure their target demographics are within an accessible range of the store. So what can you do today with competitor locations?

<h1>Targeted Local Advertising</h1>

The most intuitive use of competitor location data is for serving your ideal demographics ads using a local advertising service. Facebook ads encourage using local business advertising <a href="https://www.facebook.com/business/a/local-awareness?campaign_id=1449765931962001&placement=emadv" title="Facebook Local Business" target="_blank">near the physical store</a>, allowing an adjustable radius for serving ads. Flip the script, so that customers in your ideal demographics see your ads while in range of your competitors' stores. Try making the target radius very small, and potentially reach customers while they are at their local department store. This can be especially useful because this subset of the population already has some level of purchasing intent.

<h1>Offline Community Building</h1>

In the last few years, e-commerce companies have become more proactive about engaging with their customer community offline. For instance, <a href="https://www.combatgent.com" title="Combatant Gentleman" target="_blank">Combatant Gentleman</a> is doing a <a href="https://combatgent.com/nyc" title="Combatant Popup" target="_blank">7 day popup</a> in NYC to help foster the community and culture around its offerings. 


<h1>Putting Up a Storefront</h1>

For e-commerce companies exploring the possibility of establishing a brick and mortar presence, using competitor data to determine the most fruitful areas of expansion can be an important consideration. Warby Parker and Nasty Gal have recently established their first physical locations (in NYC and LA, respectively). While doing demographic analysis on over served locales like NYC, SF, and LA may be more straightforward, competitors' location data may be much more valuable in Cincinnati or Portland.

<h1>What Are You Waiting For?</h1>

Get started using the "business_locator" component by signing up for <a href="http://www.zillabyte.com" title="Zillabyte" target="_blank">Zillabyte</a> right now!