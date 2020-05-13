---
layout: post
title:      "JS Anxiety, pt 2: Sometimes, it's okay to hardcode"
date:       2020-05-13 06:40:33 +0000
permalink:  js_anxiety_pt_2_sometimes_its_okay_to_hardcode
---


This is a follow-up to a previous blog, found here:  https://crankums.github.io/javascripts_anxieties_shifting_gears_from_ruby_pt_1

When it comes time to actually make some stuff appear on the window, Javascript can make the process seem a little onerous. Whereas before, you might write something like: 

```
<h1>Heading!</h1>
<div class=form-box>
<form id='signup-form' class='form'>
<input type='text' id='name'>Name</input>
<input type='text' id='email'>Email</input>
</div>
```

to create:


> <h1>Heading!</h1>
> <div class=form-box>
> <h2>Signup Form</h2>
> <form id='signup-form' class='form' label='signup form'>
> <input type='text' id='name'>Name</input>
> <input type='text' id='email'>Email</input>
> </form>
> </div>


Now, you gotta go like:

```
function addForm(){
 const form = document.createElement('form')
 form.addAttribute('id', 'signup-form)
 const formBox= document.getElementById("form-box")
 formBox.appendChild(form)
}
```
...and so on.

This may not necessarily be more difficult, but it may be a hard adjustment to make if you're used to other styles of coding.  Let's have a little chat about these different styles.

The first style is hard-coded. There are other names for it, but we'll call it that for now. Every time that view is rendered, it what will appear on the screen like that. Now obviously, we can throw some logic in there, such as instance variables:

`<h2>#{@user.name}'s Profile</h2>`

and even conditionals

```if @user.logged_in?
 <h2>#{@user.name}'s Profile</h2>
  else
	<h2>Please Log In To Continue</h2>
	end```

But, for the most part, we're supposed to keep our logic out of our views. An action like that would be better taken care of in the controller. Now, of course, that means even more logic to write into the controller, and so on and so forth.

The other style is programmatic. For languages like Javascript, it renders elements to the DOM as each function responsible for doing so is called. There are classes, constants, and other entities that do these things, but for now, I'm just talking about functions. In any case, rendering things programmatically gives you a lot more control over what's rendered and how it does so under different conditions. 

The downside is that you have to have a pretty good mental picture of what you want your HTML to look like when it's all finished, or else go to the trouble of writing it out somewhere and then translating that markup to Javascript. Writing things out ahead of time is always good practice, but idea of effectively doing the same thing twice can be pretty stressful.

(How do you like to organize before writing code? Post a comment below!)

So let's think about that. Sometimes, hardcoding is nice, because it just lets you write something out, as you believe it would appear.  And sometimes, it's just the fastest way to get *something* to appear on the page. As you become more proficient in Javascript, you'll find yourself needing to do things like this below less and less, but when you're starting out, you have to do what you can to get some momentum going on your project, or else you're just sitting there, staring at a blank text editor. So, taking the steps we used in the previous two entries to check for things like outputs and contexts, let's write out some HTML like you might be used to from rails:

```
  <input type=’text’ id=’name’>Name</input>
  <input type=’text’ id=’email’>Email</input>
  <input type=’submit’>Submit</input> 
```
	
	Ok, so those are our form inputs. What's an easy way to get those into JS? How about as a variable?


```
const formInputs = “
  <input type=’text’ id=’name’>Name</input>
  <input type=’text’ id=’email’>Email</input>
  <input type=’submit’>Submit</input> 
 ”
```

Now let's toss those bits in our form creator function:

```
function addForm(){
 const form = document.createElement('form')
 form.addAttribute('id', 'signup-form)
 form.innerHTML = formImputs
 const formBox= document.getElementById("form-box")
 formBox.appendChild(form)
}
```

(What if formInputs was a function? Would that work? Why or why not?)

If I wrote the content of that *const* correctly, it should render two text inputs and a button. 
Ok, but what's the point, you might wonder? Well, like I said, this is about first steps and specifically, how to manage those steps so that you are 1) Writing code that, even if not elegant, just *works* enough for you to move on to later stages and 2) Writing code that you can *explain* when it gets reviewed and someone asks you about it.  If you're stuck, not because of lack of knowledge or lack of ability but because you're anxious or feeling overwhelmed, doing *something* is going to be better than not doing anything. You might either give up or just start copying things from lessons, other repos, or build videos because you can't start your own work, but then get paralyzed because you can't make the code you borrowed apply to your own project. So no matter how rough or how rudimentary, you have to start somewhere.

Eventually, you can refactor. Just try replacing single elements at a time with `.createElement()` calls, and see how it goes. You might even be able to create forms just by iterating the object of which the form is creating a new instance! (How might you do this? Leave a comment below). However, if it's the same form every time, why not just enure it's the same? Why not just hardcode the whole thing, if it's not going to be dynamically generated?

Ask youself, what changes in this thing I'm rendering? If the answer is nothing, why might you not want to hardcode it? Or alternately, how difficult might it be to build my code around this thing not being dynamically altered? Again, this isn't "the $5 dollar thing is better than the $30 thing". That's not it. The point is that if you've gotten this far, you already have some base of coding knowledge, so start with what you know!

And it's great practice for React (I said it)

-Craig
