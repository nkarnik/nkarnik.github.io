---
layout: post
title: Mapping Tech (with) Bubbles
excerpt: "I mapped the locations and sizes of startup funding rounds with their locations in major cities"
modified: 2014-11-23
categories: articles
tags: [zillabyte,python,crunchbase,startups,web scraping,web crawling]
comments: true
share: true
---

*(Note: I originally published this at the [Zillabyte blog](http://blog.zillabyte.com/2014/11/23/mapping-tech-bubbles/))*

A time lapse of venture funding rounds (by date announced) for a sample of 1000 companies in SF:

<iframe width='100%' height='520' frameborder='0' src='http://zillabyte.cartodb.com/viz/f4353cee-73a6-11e4-aabd-0e9d821ea90d/embed_map' allowfullscreen webkitallowfullscreen mozallowfullscreen oallowfullscreen msallowfullscreen></iframe>


...I wonder why rent in SOMA is so expensive.

<h1>Mapping Lots of Bubbles</h1>


I have very few opinions about whether tech companies have been chronically overvalued by institutional venture capitalists. However, it is hard to ignore that quite a bit of venture capital has been poured into a wide array (though eventually concentrated) set of companies that did not exist (1, 5, 10) years ago. Many of these companies have sprouted up in the densest parts America's urban centers. To that end, I am analyzing how the most recent tech boom has impacted America's cities. 

I collected data about venture-backed companies using the <a href="http://www.crunchbase.com" title="Crunchbase" target="_blank">Crunchbase</a> API, which allows for a limited number of requests. For my purposes, the relevant information for each company included: company name, total funding raised, street address, city/state, and (for later use) API ids for all funding rounds. My goals were three-fold: 1) Map each company within a city with bubble size corresponding to the total funding raised. 2) Map the "acceleration" of venture funding with a time-lapse map. 3) Maybe compare trends between cities.

In these maps, bubble sizes correspond to the total amount of funding raised. Again, this is a semi-random sample of about 1000 companies, so please don't feel bad if your/your favorite startup isn't on this map.

All of the maps in this post were rendered with the use of <a href="http://www.cartodb.com" title="CartoDB" target="_blank">CartoDB</a>, an excellent piece of map visualization software. I highly recommend it for your next project.


A static (stylized) map of startup locations by city block in San Francisco:

<iframe width='100%' height='520' frameborder='0' src='http://zillabyte.cartodb.com/viz/82f0f08a-73a8-11e4-ae1c-0e4fddd5de28/embed_map' allowfullscreen webkitallowfullscreen mozallowfullscreen oallowfullscreen msallowfullscreen></iframe>


<iframe width='100%' height='520' frameborder='0' src='http://zillabyte.cartodb.com/viz/d29ab428-73a7-11e4-8c2e-0e018d66dc29/embed_map' allowfullscreen webkitallowfullscreen mozallowfullscreen oallowfullscreen msallowfullscreen></iframe>

<h2>Building the App</h2>

Using <a href="http://docs.zillabyte.com" title="Zillabyte Documentation" target="_blank">Zillabyte</a>, I was able to streamline my data processing. I started with a stream of company names from each city, and tried to make valid crunchbase API requests based on how company names are represented in the crunchbase database.


{% highlight python %}
import zillabyte
import crunch
import csv

def prep(controller):
  return

def nt(controller):
  infile = open("sfvalid.csv", "rt")
  company_list = []
  c = 0

  try:
    reader = csv.reader(infile)
    for row in reader:
      controller.emit({"company" : row[0]})
      c += 1

      if c > 5:
        print c
        controller.end_cycle()

  except:
    print "Error"

  if c > 20:
    print c
    controller.end_cycle()


def execute(controller, tup):

  company = tup["company"]
  data = crunch.getOrgRow(company)
  #tup = (org, funding, address, lat, lon, funds)
  print data

  controller.emit({"company":data[0],"funding":data[1],"address":data[2],"lat":data[3],"lon":data[4],"fund_paths": data[5]})
  return




app = zillabyte.app(name = "startups")

app.source(name="startuplist", next_tuple = nt, end_cycle_policy="explicit")\
   .each(execute = execute)\
   .sink(name = "startups", columns = [{"company":"string"},{"funding":"integer"},{"address":"string"},{"lat":"float"},{"lon":"float"},{"fund_paths":"array"}])
{% endhighlight %}


You may have noticed that I have a custom crunch module that makes my API requests. The relevant contents follow:

{% highlight python %}

import requests
import json
import csv
import sys
import time

key = "YOURKEYHERE"

def getTotalFunding(orgjson):

  funding = orgjson["data"]["properties"]["total_funding_usd"]
  return funding


def getLatLong(street):

  street = street.replace(" ", "+")
  request_url = "http://www.datasciencetoolkit.org/street2coordinates/" + street
  # ALSO Need city, state at the end

  jreq = requests.get(request_url).json()

  lat = jreq[jreq.keys()[0]]["latitude"]
  lon = jreq[jreq.keys()[0]]["longitude"]
  return (lat, lon)



def getOrganization(org):

  request_url = "http://api.crunchbase.com/v/2/organization/" + org + "?user_key=" + key
  jreq = requests.get(request_url).json()
  return jreq


def validate(orgjson):

  if len(orgjson.keys()) < 2:
    print "invalid"
    return False

  return True


def getLocation(orgjson):

  location = orgjson["data"]["relationships"]["headquarters"]["items"][0]

  latitude = location["latitude"]
  longitude = location["longitude"]
  city = location["city"]

  if (location["street_1"] is None) and (location["street_2"] is None):
    return False

  elif (location["street_1"] is None):
    street = location["street_2"]

  elif (location["street_2"] is None):
    street = location["street_1"]

  else:
    street = location["street_1"] + " " + location["street_2"]

  return street

def getFundingRounds(orgjson):

  fundingRounds = orgjson["data"]["relationships"]["funding_rounds"]["items"]
  rounds = []
  frs = []

  for fundingRound in fundingRounds:
    rounds.append(fundingRound["path"])

  return rounds



def getOrgRow(org):

  try:
    jorg = getOrganization(org)
    if validate(jorg):

      funding = getTotalFunding(jorg)
      address = str(getLocation(jorg))

      lat, lon = getLatLong(address)

      rounds = getFundingRounds(jorg)
      funds = {"rounds": []}
      for r in rounds:
         funds["rounds"].append(str(r))

      tup = (org, funding, address, lat, lon, funds)

      return tup

  except:
    print "error"
{% endhighlight %}


In order to geotag startups latitude and longitude by street name (and implicitly state and country), I made an external API request to datasciencetoolkit's Georeferencer, which can be called as follows: 

request_url = "http://www.datasciencetoolkit.org/street2coordinates/" + street + "%2c+San+Francisco%2c+CA"

Making an http request to this url (with your street embedded) should return a fairly accurate lat, long coordinate for each company. This isn't a perfect georeference tool; in fact, some of the coordinates are mislabeled (i.e. there is no startup located on Treasure Island).


<h2>Other Cities</h2>

So my stretch goal for this exercise was to analyze other cities startup trajectories. The world doesn't revolve around SF, of course. Palo Alto is an interesting case, because there are a ton of startups there, but the area is significantly more suburban:

<iframe width='100%' height='520' frameborder='0' src='http://zillabyte.cartodb.com/viz/d0033926-7396-11e4-bcac-0e4fddd5de28/embed_map' allowfullscreen webkitallowfullscreen mozallowfullscreen oallowfullscreen msallowfullscreen></iframe>


In Palo Alto, most of the startup activity is concentrated around the two mass transit Caltrain stops, the downtown Palo Alto station and the station at California Avenue. Access to transportation is a critical component of how people/businesses are concentrated, so this should not be a huge surprise. 


<b>NYC</b>:

Time lapse of funding rounds in NYC (limited to Manhattan). Interesting to note that while funding definitely picks up in pace after 2009, the overall growth in startups appears more steady than the relative explosion in San Francisco. Of course, this is all very hand-wavy, and as an engineer/analyst I should really quantify this better:

<iframe width='100%' height='520' frameborder='0' src='http://zillabyte.cartodb.com/viz/cce44e00-73ab-11e4-b546-0e018d66dc29/embed_map' allowfullscreen webkitallowfullscreen mozallowfullscreen oallowfullscreen msallowfullscreen></iframe>



<b>Austin</b>:

Static sample map of Austin's startup scene. I have never stepped foot into Texas and know nothing about Austin, but it is interesting to see businesses spaced out further in a city.

<iframe width='100%' height='520' frameborder='0' src='http://zillabyte.cartodb.com/viz/ed5f571c-73ae-11e4-8c2e-0e018d66dc29/embed_map' allowfullscreen webkitallowfullscreen mozallowfullscreen oallowfullscreen msallowfullscreen></iframe>



<h2>Takeaways</h2>

After spending a few hours reading about the history of American cities, I can confirm that I am woefully unqualified to make most sweeping claims about how different cities' startup communities have evolved over the last 12 years. Cities across America (and the world) have very different histories, and the next step should be to quantify urban movement (population growth) and development relative to the rate of venture funding/startup growth in each city. This way, we can build off this work to analyze the following: How does the acceleration of startup growth compare across cities? How much are changing demographics in large urban areas correlated with the latest tech boom? Which cities are most primed to be the "up and coming" startup hubs?