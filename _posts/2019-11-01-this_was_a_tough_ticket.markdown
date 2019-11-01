---
layout: post
title:      "This was a tough ticket"
date:       2019-11-01 15:29:17 +0000
permalink:  this_was_a_tough_ticket
---


#### Please excuse the placeholders, I'll replace them with screencaps or code snippets shortly

I've spent a lot of time in bike shops and at least 8 years working in them, so it was a natural choice to try to make a work order application like I'd use in a shop for repairs. A natural choice, and unfortunately, not my first one, but that's a different story. 

The application would be a non-customer facing type, so the people logging in and using it are the shop staff (such as mechanics, managers, etc). This also means that most of the resources are at the disposal of all of the users. To expand on that, there's ownership of a ticket in that a particular person may have written it, but anyone can edit it. This is very important to the operation of a shop when tickets need to updated, say, extra time added or the labor changed. 

The thing about a work order is that it combines several distinct entities into a single item. That's where things get complicated.

The models in this app are:

User,
Customer,
Ticket, <= what we're calling the work order
Bike

There's a couple others. There's an Admin model, which mainly exists as a method container. There's a Calendars resource, as well, for creating different calendar views to show when service is scheduled. We can circle back to these, but let's talk about our four "primary" models.

User: the person using the app. In this case, it would be a shop employee. This means that while they have an identity within the app, the information stored about them isn't particularly important. This is different than how a lot of other people I've seen at this stage design their apps, in which big-U User is the center of the app's universe. The user `has_many: customers, through: :tickets` and `has_many :tickets`. 

Customer: the person for whom we're writing the work order. We have a little more information about them. They also have a fairly complex relationship position. The customer `has_many: bikes`, `has_many: tickets`, and `has_many: users, through: tickets`. 

The Ticket model is where the work order is written. This sits at the center of the app, because the ticket needs to have information about the customer and their bike. It also helps to have at least some info about the user passed in at some point. Therefore, the ticket `belongs_to :customer`, `belongs_to :user`, and `has_one :bike`. Why doesn't the ticket `has_many :bikes`? Because you can only work on one at a time, but more importantly, it saves some work. While at no point should you be multi-assigning bikes to your ticket, it also means that any methods you call to return data about the bike won't have to be prefaced with searcing for which bike you want. It also means that if you assign a bike to the ticket through `#ticket.bike=`, it simply replaces the bike instance instead of populating a `bike` array. 

Finally, there's the bike model. We don't want an association to the user, because the user doesn't own the bike, they only interact with the bike through their association with the ticket. We also don't want to spend time assigning, clearing, and reassigning bikes to the user just because they're working on them. As a sidebar, you could make a `:working_on` attributed that is cleared to `none` or `nil` when the ticket is `complete` or `canceled`. A benefit to this is that you could use the attribute to determine who is open to assignment and assign the ticket to them. For the purposes of this project, I decided to not to create any assignment attributes, as it's pretty common for mechanics to take tickets by their status ("Open", "In Progress" etc), rather than wait to be assigned them. 

Back to the track, the bike model takes in mostly indentifying info about the bike. I've never seen a service form contain more about the bike than its make, model, and color ( and maybe its size). That's because there's just too much information. It is tempting to get very detail oriented and make lines for everything from tire size to cable routing, but in the end, it's just not important. The mechanic is exected to be able to figure the other details out, but they need to find the bike, hence the make (which is, with extremely limited exception, always the biggest decal on the bike), the model (which in my schema is named `model_series`, to prevent conflict with an existing `model` method), the color, and the size (which is helpful when there are four black Specialized Sirrus bikes hanging in your shop, aka, every July).

So what do we do with all this? Well...
The most brute force solution would have been to create a form and give Ticket all of these attributes, but I think as students and as a society, we're past that. Alright, alright, what then?

Here's where some nesting comes in handy. You'll need to do a little conditional work, figuring out which params are passed in and which you need to look for.

Let's toss a search bar in there. 

[search_bar_screen]

We should make a search method for this to work

[search method]

By all means, be very specific and it'll deliver. There's no reason to put in a custom Google search bar to search one model's table.

It doesn't have to be too complicated, you can have it search by any attribute you have. Find the customer in the `'/index'` page, follow the link to their `'/show` page, and from there, you can make a new ticket with the customer params passed in. 

Also, let's have a form that takes all new info, so you can make a customer, bike, and ticket all at once. This is going to be the most common way you add a custie (<== industry term) and their bike to your database.  The other way would be through a sale, but this app isn't set up for cashiering at this time. 

In our form, our little buddy `fields_for` is crucial. As a note, we were told NOT to use `form_for`, as apparently it's being deprecated. So I build everything with `form_with` whenever possible. Despite the name, `fields_for` is all gravy. If this is incorrect, I've got some work ahead of me. Anyway, `fields_for` will pass your `params[:tickets]` with your chosen attributes nested in, like `params[:tickets][:customers]`. (Another note: `fields_for` model has to be plural! Now back to our programming). Now I have those params being passed through in a discrete and manageable way, as opposed to manually specifying individual params in your `#create` or `#build` methods. 

BUT...how?

We know how to make strong params. In your argument field, pass in the `fields_for` hash, like so:

[fields_for_hash]

AND THEN!

there's a handy little method called `#except`, which allows to you leave out passing in certain params. With everything wrapped up tidy in the hash provided by `fields_for`, we can just ask it to:

[params_except]

To prevent the error being thrown when the `#create` method tries to pass in an attribute that isn't there.

So nested forms aren't so bad, right?


