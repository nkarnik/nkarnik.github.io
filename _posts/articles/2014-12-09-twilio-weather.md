---
layout: post
title: How to Build a Global Weather Alert System in Under an Hour
excerpt: "I decided to have some fun with weather data and Twilio to gently inform my landlord when the rains are coming"
modified: 2014-12-09
categories: articles
tags: [zillabyte,python,weather,Twilio,web scraping,web crawling]
comments: true
share: true
---

California is in the middle of a historically extreme drought. While this isn't news to anyone, San Francisco has seen an unexpectedly large amount of rain in the last couple weeks, causing problems for many residents.

Personally, my building has had major leaking and flooding issues each of the last 3 weeks. After getting fed up over the overall lack of preparation and response, I decided to have some fun with weather data and <a href="http://www.twilio.com" title="twilio" target="_blank">Twilio</a> to "gently inform" my landlord when the rains are coming.

<img src="https://s3.amazonaws.com/uploads.hipchat.com/50986/345019/VMKpwNzdwrUCfJO/upload.png" alt="Zillaweather Workflow" />

<h1>Sneak Preview: the Crux of the App</h1>

{% highlight python %}
import zillabyte
from zillaweather import TwilioEach

app = zillabyte.app(name="zillaweather")

#source from list of world cities
stream = app.source("all_cities")

#call component to find current weather condition at each city
stream = stream.call_component("tomorrow_weather")

#Use current weather and dictionary of cities : numbers to send a SMS with Twilio
stream = stream.each(TwilioEach, name="Prepare_Twilio")
stream = stream.call_component("twilio_text")

stream.sink(name="weather", columns = [{"to":"string"}])
{% endhighlight %}

That's seriously it. In full disclosure, this is what the <a href="https://gist.github.com/nkarnik/1d40b230370c72830ec0" title="Zillaweather" target="_blank">entire app</a> looks like without modularizing the TwilioEach class. All things considered, maybe 30 lines of code total. How? All because of the power of components.


<h1>Sending Twilio messages with Zillabyte</h1>

If the 3 day weather forecast calls for a large amount of rain, I can probably just check an app and let my landlord know in advance. But, hold on... Zillabyte is a service for distributed computing... so what's the fun of being limited to weather data from one city? I want to help tenants notify their landlord all across America (even the whole world!) about upcoming local weather conditions. I am better served using Zillabyte to query and process data from almost every weather station in America and an automated service like Twilio to serve the notifications. 


First, I wrapped all of my Twilio rest API calls inside a Zillabyte component. After registering this component, I can call this component at will. The component simply sends a message from a Twilio number about the current weather conditions in a city to a phone number subscribed to that city.

{% highlight python %}
import zillabyte
from twilio.rest import TwilioRestClient
 
def send_text(controller, tup):
 
  # Register twilio client
  twilio_client = TwilioRestClient(tup["twilio_sid"], tup["twilio_auth"])
 
  try:
    # Send message to recipient and wait 1 second (twilio SMS rate limit)
    text_body = "It is going to be " + tup['condition'] + " in " + tup['city'] + " tomorrow!"
    twilio_client.messages.create(to=tup["to"], from= tup["from"], body= text_body)
 
    # Emit the recipient back to the stream...
    controller.emit({"to" : text_to})
    time.sleep(1)
 
  except:
    pass
 
component = zillabyte.component(name="twilio_weather_text")
stream = component.inputs( 
  name = "input_stream",\
  fields = [{"twilio_sid" : "string"}, {"twilio_auth" : "string"},\
            {"from" : "string"}, {"to" : "string"}, {"city" : "string"},\
            {"condition" : "string"}]\
  )
 
stream = stream.each(send_text)
stream.outputs(name = "output_stream", fields=[{"to" : "string"}])
{% endhighlight %}

Fork <a href="https://gist.github.com/nkarnik/311d3a9bd851186ee4b3" title="Twilio Component" target="_blank">this component</a>  on github.

<h1>The Entire App</h2>

After registering the component, the <a href="https://gist.github.com/nkarnik/1d40b230370c72830ec0" title="Zillaweather" target="_blank">remainder of the Zillabyte app</a> is straightforward to write.  The app uses a pre-registered component called "tomorrow_weather", which will determine the expected forecast for tomorrow for a given city. It then sends rain notifications to phone numbers registered by city via the Twilio "send_text" component.



<h1>Why Mix Data and Notifications?</h1>


The primary utility and accessibility of "big data" stems from the ability of data systems to provide meaningful insights to humans. Certainly, many times large-scale data systems are built to feed some dynamic model on the back-end (such as an Ad model on a social network service), but under certain conditions, notifying humans is necessary. For example, if your Ad model is performing particularly poorly (users aren't clicking because they aren't being served relevant ads), your data science team might want to receive a text notification in case they're out to lunch.