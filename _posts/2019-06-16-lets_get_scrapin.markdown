---
layout: post
title:      "Let's get scrapin'"
date:       2019-06-16 18:12:39 +0000
permalink:  lets_get_scrapin
---



Our cohort’s project to finish of our segment of Object Oriented Ruby lessons is to build a web scraper. 

A web scraper digs into the guts and bits of a web page and extracts the tasty morsels inside, and most importantly, to perform the extraction without grabbing a lot of noisy HTML artifacts or unviewable media, like pictures or videos. I work for a marketing intelligence company and scrapers are necessary in order for us to build predictive models of customer activity for our clients. My particular job works WITH data provided to us by numerous scraping and transcription tools (with some pretty darn awesome applications developed in-house, I’ll add), but I don’t use these tools directly, so it was pretty exciting to get under the hood and see how these things operate.

Now, excessive preamble is for recipes you find online, so let’s have a look at my project.

**The Website:**

I based my choice of site to scrape based on both a company and a product I know very well. Velocity USA is a manufacturer of bicycle rims, hubs, and a few other parts. They have an excellent variety of products and are generally a good go-to choice for rims between their pricing, variety, and quality. The variety is very important as a guardrail against hard-coding anything. Bicycle rims have consistent features, but different values for each of these (such as diameter, tire width, and a few others). There isn’t going to be a rim without spoke drillings, but those drillings can vary wildly. Therefore, I had a list of features to eventually arrive at, but I would need to scrape in order to obtain the specific features. Another reason I chose this site is my familiarity with the product. If the `rim_size:` attribute for the Atlas 26” rim came out to say `622`, I would immediately know that is incorrect (the rim size of a 26” rim is 559mm, if you were curious). Finally, the organization of the site made this an easier project. The data I needed was in a fairly tight group, giving me a little more control over the scraping tools for my first foray. It also provided me with a nice bit of direction to organize the flow of my CLI, which I’ll go into, further below. 

Ok, so there’s the “why?”, now how about some “what?”
The tools I used to perform this operation were Open-URI and Nokogiri. Open-URI is part of Ruby’s standard toolset, so all you have to do is `require` it in your class models and `environment.rb` folder. Among other things, the gem allows you to open a web page and create a temporary file through which you can comb and parse bits of data for your purposes. Open-URI doesn’t actually let you do the parsing, though, at least not how I used it. For that, we used another gem called Nokogiri. 

This would be a good spot for some of those obligatory screenshots:

![Gemfile](https://i.imgur.com/Ide26sG.png)
You don’t need to require it in the Gemfile, because it’s already part of Ruby’s standard toolset

![Scraper reqs](https://i.imgur.com/L9i3VvD.png)

You generally only need to require gems in files where you’re going to use them.

![environment.rb](https://i.imgur.com/htZIbDh.png)

The nerve cluster. I honestly required a lot of help getting the gears to turn on this project, and getting things like your Environment file, `bin/` files, and other segments functioning properly are to, of course, finishing your project, but also for running it, testing it, and making sure there are no time bombs and trapdoors waiting for you when you try to present your project to someone.

**Nokogiri**

There’s an ocean of information on the particulars of Nokogiri out there, so I’ll keep this brief. Nokogiri is a tool, named after a type of Japanese handsaw with very fine teeth, that allows you comb though and parse specific elements of a web page. You can perform its functions on HTML, XML, and other mediums. I used the HTML functionality for this project as this was how we were taught to use the gem in our coursework at Flatiron. It is not so much that we were not permitted to use other methods, or even other gems entirely; rather, I wanted to work with the basic set of tools that I was given, so that I had a better understanding of the processes at work. This is a pretty typical teaching method: when you learn arrays, a coding course might tell you to reverse the array without using `.reverse` or to gather elements into an array without using `.map`. 
Now back to the process. The sequence of using Open-URI and Nokogiri go something like this:

```
html = open(url)

doc = Nokogiri::HTML(html)
```
From there, you can do things like 

```
page = doc.css(#div.content).map {|el| el.text}
page.each do |name|
	puts “#{name.first}”
end
```

It’s important, as always, to save your iterated or captured data in variables so you can call methods or simply return at a later point in your code. This seems incredibly basic, but it’s a really easy thing to overlook, especially if you’re in Pry and you can’t repeatedly run the file without re-entering all the code. Specifically, it’s better to save the data from `open(url)` rather than run Open-URI directly from Nokogiri’s argument space. At least, that was my experience with it. It’s also common practice to save your starting URL as a constant, which is conventionalized in Ruby as all caps (“BASE_URL”). A capital letter at the beginning of a variable signifies a constant, and all caps is just a way of signaling that it’s a constant rather than a class. 
Now what are we looking for in our web page to scrape? Let’s open up those dev tools and have a look (`ctrl+shift+i` or F12 ”*I use Windows, yes I do*). Here’s all that freaky stuff we get when you mean to hit Home or 0 or something and oh god my computer is broken, what did I do, ok whatever. 

![applications tab](https://i.imgur.com/XWRFUsp.png)

My elements, like I said before, were all in one place. That’s handy. From here, I can grab the links to the next level down and the names of each of the applications. I decided that the flow for my gem should go like this :

***Greeting > Menu > Application > Rims > Specs > Final Selections***

I wanted to make this like a consumer app, because, hey, I’d like to make consumer apps. So, I built in a bunch of greetings and questions to direct how this gem is used. There are some features left out for a couple reasons, which I’ll get to at the end.

![application-rims](https://i.imgur.com/ENEn9zd.png)
Oh no, totally missed a part of how this site is organized. The project requirement was to scrape two levels down. It looks like I’ll need to scrape three to actually get the specs of the rims and not just a lift of their names! So, I scrape the `rim-applications/’application’` page, like I did with Applications, earlier, creating a hash of names and URLs. I fed these key-value pairs into an iterator and created a bunch of new Application instances, and I’ll do the same with Rims. Then, for each Rim instance created, I’ll scrape its page, create hash for specs and feed those into my Rim instances, using the handy `.send` method (which will likely be my favorite given how much time it saves me populating attributes).

Let’s scrape actual product level:

![rim-specs](https://imgur.com/ELJ4MUq)

This is handy, everything I need is in `#prod_left`. But wait a minute, when I parse out each of these specs, I can’t seem to just get the values, which I wanted to save to a hash to feed in to the attributes of my `Rim` class. What’s this `#text` stuff on the other side of `</span>`? That’s annoying, I either have to take the whole thing, which would include the title, or I can’t seem to get it, at least not with what I’m using. However, when I think about it, there’s actually nothing wrong with having both. When it comes time to display the specs for the rim selected, it would save me from having to input the keys of the specs over again. I can just say `puts “#{:attribute}”` and get the whole bit! That’ll save some time. 

Now we do something different for spokes and colors. Rims can have different spoke drillings and come in different colors. Here’s the first lump I’ll take: Color choices and spoke drillings are paired. I also left out machined versus non-machined sidewalls. That means a rim can be green, non-machined sidewalls, 32-spoke, but only if the manufacturer offers this SPECIFIC combination. The problem is that this gem is getting big and I can’t risk it getting too unwieldy. This manufacturer DOES do custom anodizing and drilling, so I’ll build the remaining portion with this in mind. You want 18 spokes in blue, machined sidewalls? Sure, let’s just do that! By compressing spoke counts and color choices into two lists, this streamlines the menu and control flow quite a bit. Gotta control that scope creep, and I’ll do it whenever I can!

Ok, now here’s the other lump: the search function. I had to eliminate “See a list of rims by name” from the starting menu, because that would have required yet ANOTHER scrape, and I was getting worried about how many dips into data I was taking at this point. I already have three scrapers in this gem as it is. My “Search” option had a similar issue: no lists were populated yet. But let’s rung what ya brung. I rephrased my “Search” option to look through previously viewed items (with the hidden bonus that it actually goes through any previously LISTED rims, not just viewed. I might change that phrasing). A future fix would be to scrape through the `product/rims#models-tab` to populate lists by name, then scrape those accordingly for the specs. A very important feature is to build a `self.find_or_create_by_name(name)` class method and extend it so as to keep from continually accumulating a mighty list of new instances. The search function also lacks flexibility. Currently, it can convert the input to string, but it needs to be an exact match, so copy and pasting is preferred over typing. 

Those are the big details regarding my CLI Project, so here’s to hoping it goes over well when it’s time to present it. Thanks for coming to my CRAIG talk.
