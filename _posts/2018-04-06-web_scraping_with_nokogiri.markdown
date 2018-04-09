---
layout: post
title:      "Web Scraping With Nokogiri"
date:       2018-04-06 03:48:51 -0400
permalink:  web_scraping_with_nokogiri
---


After learning how to scrape websites for data I realized how important it is in web development. So many app ideas came to me after learning this new skill and it’s very easy to learn. All you need is a basic understanding of HTML, CSS and Ruby to get you going. In this post I’m going to set you up with the skills you need to retrieve content from any website.


## The Setup

First in your scraping file lets call it “web_scrape.rb” you need to `require “nokogiri” `, `require  “open-uri”` atop of your file and install on the command line with `gem install nokogiri`.
The “open-uri” that we required allows us to extract all of the HTML from the page we want to scrape. This can be done with XML documents also. We can do this using the `open()` method, thanks to open-uri.

``` 
require “nokogiri”
require “open-uri”

html = open("https://www.moving.com/tips/the-top-10-largest-us-cities-by-population/")

```

Now that we have required our methods and got all of the HTML off of the page let’s find out a bit about Nokogiri. So, at this point  in our `html` variable we have something like this, but bigger.



![](http://res.cloudinary.com/zacwillmington/image/upload/v1523000575/Screen_Shot_2018-03-19_at_11.16.53_AM_kmigcy.png)




## Parsing

There is a lot going on here and its hard to read. Which brings us to Nokogiri’s role in this situation. Nokogiri uses methods to change data to in order to work with it. It parses the massive string of HTML or XML in to individual nodes so we can reach in and access the information we want. 


``` 
require “nokogiri”
require “open-uri”

html = open("https://www.moving.com/tips/the-top-10-largest-us-cities-by-population/")

doc = Nokogiri::HTML(html)

```


## Scraping

The new line above is basically converting that super long document of HTML to a bunch of nodes. Take note that if you are working with a XML file just replace the HTML with XML. Now that we have the new doc variable that Nokogiri parsed for us, we can use a cool method to get what we want out of the page. As far as the HTML of the web page, the only thing that has changed is the how we see it, so we can use our developer tools in chrome to see what we want to get.  


![](http://res.cloudinary.com/zacwillmington/image/upload/v1523000584/Screen_Shot_2018-03-19_at_11.17.24_AM_flswos.png)



As shown above a we want the text from:

```
<h2>
	<strong>#1 New York City, NY</strong>	
</h2>
```
 
By looking at the layout of the page, I’ve noticed all of the cities have the same elements. Now we can grab all the `h2` and `strong` elements text. This is how we get it.

```

require "nokogiri"
require "open-uri"

html = open("https://www.moving.com/tips/the-top-10-largest-us-cities-by-population/")

doc = Nokogiri::HTML(html)

cities = doc.css("h2 strong")

  cities.each do |city|
      puts city.text
 end
 

#Output
=> #1 New York City, NY
=> #2 Los Angeles, CA
=> #3 Chicago, IL
=> #4 Houston, TX
=> #5 Philadelphia, PA
=> #6 Phoenix, AZ
=> #7 San Antonio, TX
=> #8 San Diego, CA
=> #9 Dallas, TX
=> #10 San Jose, CA


```


Notice that the strong element is nested so I have mentioned `h2 `then strong to specify only strong tags that are nested in `h2` elements. Let’s say you only want to select the first `h2 strong` text. If you are lucky and the `h2` or `strong` element has a unique class, ID or title name you are able to select the element you want with the `.css()` method by using `doc.css("h2 strong”).first` to get the first node in the node set.


When selecting text using the class or ID attributes you must remember that if you want to select any of theses attributes you need to call them in the method just like you would in css
stylesheet e.g.  .class_name or #id_name.


`.search()` is another of Nokogiri’s methods that makes it effortlessly easy to search for HTML elements used in the same way as the `.css()` method. You can check out what else Nokogiri has to offer on "http://www.nokogiri.org/"

If you would like to see this technique in action in an application, I’ve built a simple CLI app (Command Line Interface) that scrapes the California National Parks website for info on all the major Parks in California. Have a look at ”CLI app" .

![](http://res.cloudinary.com/zacwillmington/image/upload/v1523000575/Screen_Shot_2018-03-19_at_11.16.53_AM_kmigcy.png)

![](http://res.cloudinary.com/zacwillmington/image/upload/v1523000584/Screen_Shot_2018-03-19_at_11.17.24_AM_flswos.png)


