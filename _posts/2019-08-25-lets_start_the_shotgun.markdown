---
layout: post
title:      "Let's start the Shotgun!"
date:       2019-08-25 22:54:54 -0400
permalink:  lets_start_the_shotgun
---


Greetings, kitty kats. 

My post today is going to talk about an important part of my Sinatra project. Namely, what do we have to do when creating a new object to make sure that it gets associated to a previously made one?
Well!
First, let's talk about ActiveRecord. Active Record is how we are going to create our model and all of its attributes. So maybe you remember this:

```
class Object
  attr_accessor :attrib1, attrib2, etc
	
	 @@class_variable = []
	 
	 def initialize(attrib1, attrib2)
	   @attrib1 = attrib1
		 @attrib2 = attrib2
		 @@class_variable << self
		end
		
		def self.all
		  @@class_variable
		end
		
		# and so on...
		end
	```
		
This is how we created our object, and if you're learning to code, your interactions with this object or how this one plays with others like could be the the most you've seen your code *actually* do. Meaning, you write some stuff, you run your tests and see if it's passing or not. Hopefully, you've been taking a look under the hood with utilities like `binding.pry`, consoles, or `puts` commands in your code or terminal, and, if you've made a CLI, then you've been able to view responses to your inputs in your terminal. 

There's an issue here and that is that everything you've created needs to be rebuilt pretty much every time you fire up your project. Maybe you need to write in the attributes, or you need to have your scraper object run to retireve the information you need and hope the page is active, taking requests, and hasn't changed. That sounds like work, and it's time and resources being spent when you could off doing greater things! All this is to stay, what we need here is a way for our data to persist.

Here comes Active Record. I'm going to skip over SQL for this blog entry, except to say: there is SQL, which exists to allow a user to create identifiable tables of data, and then be able to request that data back again. Not only that, but to do so in a format that it's easy to read, filter, and arrange. Finally, I need to be able to make updates and delete entries. Now all of this sounds great, but have you ever had to write SQL queries?

```
SELECT row, other_row 
FROM table
WHERE filter_item = "attribute" 
ORDER BY desired_order;
```
		 
Not the worst thing, but a bit wordy, and can add quite a bit of lines to our code. To make a long story short, this is where ActiveRecord comes in. Active Record is a suite of utilities that helps *automate* certain processes to save time and make more advanced processes possible. There are a huge amount of methods built in, which allows for flexible, dynamic code to be written. Not only that, but it helps us create, manage, and organize *persistent* objects in memory, so that we don't have to sit there and rebuild this stuff every time you want to test or build your code. We'll use ActiveRecord with some valuable assistance from a tool called Rake. The latter is an extremely valuable automation tool, in that, among other things, allows for the creation and modification of database tables that you can then query and manipulate using Active Record tools. I'll talk more about Rake another time, but let's get back to the subject of this entry.

Fast forwarding, I've built my objects to inherit Active Record's methods, I've used Rake to create my database migrations and now I'm ready to create some objects and pass them from model to model, each maintaining its relationship through the last, whether it `belongs_to:`, `has_many`, or `has_many_through`. So how does Active Record make maintaining these object relationships easier? Well, let's start with the fact that the relationships I just mentioned before, along with many others, are methods you can actually call in your object (in the case, the Model) class. So an `Author` class `has_many: books`. This adds a database row to `Authors` along with a self-named method called `books`. Therefore, I could call `Author.books.all` and get a list of all the author's books that we might have saved in the database (obviously, it's not magic, you do have to make sure it makes its way in somehow.)

It gets even better. We might be familiar with `Object.new(args)` and `Object.create(different_args)`, the latter including a `#save` method built in. Active Record then gives us a `#build` and `#create_` methods. This means that I could do:

```
author = Authors.all.last 
# in this case, grabbing the last 'author' object I created
book = author.build(params[:attribute])
```

The `book` object created will then have an `author_id` attribute added to it. Now I can find a book with:

```
book = Books.find_by(author_id: params[:author_id])
```

You may have noticed two things. One, that the argument used to create or find my `book` was a symbol, and two, the word `params`.  Params, short for parameters, not quite magic, but rather a cleverly written getter method that retrieves certain pre-designated attributes. You can call `params` as an argument in itself, which adds a big hash, including attributed you may not actually have a place for. This will throw an error. Instead, you can call them individually, so I can call:

```
book = author.build(title: params[:title], isbn: params[:isbn])
```

The new book will have the named attributes, and `belong_to` an author, which we can quantify by the presence of its `author_id` attribute. In turn, we can also observe this happening in the other direction by calling:
```
author.books
=>[#an array of books assigned to the author]
```

The `create_` method also allows this, in that if a book `has_one` or `has_many: characters`, I could create a character in a book entry like so:

```
character = book.create_character(attributes: params[:attributes])
```

Again, this will create a `character` object, associated with a `book` object. It also adds readability, as we are unambiguously creating a character in this line. Finally, depending on how you might have prepared the arguments you'll pass into your `create_` method, you can make your code a little less unwieldy and thus easier to read and edit down the line. 

Most importantly, we've managed to create an `author` to a `book` to a `character` class, and with the help of persistant data that we're able to continously find, read, edit, and assign, all of these items are linked together though a series of `id` attributes or are specifically listed in the object's scope (like our `author.books` method). This will help our coding immensely when it's time to make code that's seen outside of the terminal. Once we start to make responsive websites that will maintain huge amounts of data, we will need to be specific about which items we're calling and, for both readability and security purposes, only certain information is displayed. 
