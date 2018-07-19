--- 
layout: post
title: "Small but irritating things - Git."
author: "Author"
comments: true
tags : [Programming, Git]
---

I tried to retrieve a stash by name in Git. Honest, thats all I did. 

Yet here I am.

Software is full of small and intensely irritating things.  Repeat after me - _Software is full of small and intensely irritating things._ 

You will encounter it again and again, in good software, bad software, throw away software and never used software _(for obvious reasons)_. And you will encounter it in the best of software too. 

Which, from now onwards, I will be documenting. 

And today is a Git day.

## Short historical background on GIT.
Lets digress - 
- The person who wrote Git was, or rather, is the most famous programmer alive and quite possibly, the most respected as well. Name's Linus Torvalds.
- Git again is like a cross between the Mona Lisa and the Swiss Army Knife (_the sort which Schwarzenneger carries into a Predator jungle and builds a seven storeyed Harrods with_). Its an engineering marvel; Considering the distributed part, its a conceptual marvel; its incredibly robust; It's blazing fast and its very versatile. 

So given these ancestors and these antecedents, you would assume that Git is flawless - right? And its venerable primary author and the plethora of talented programmers who have lovingly grafted their art onto the foundations laid by the erstwhile venerable one, have paid complete attention to both functionality and ease of use which is why you have a billion flags (options) with each command, and there are at least 7 ways to do any one thing (exaggerating only a bit), and its rock solid - correct? Well, yes, its all of that, but inspite of all that *<strong>try, just try, to retrieve a stash by name</strong>*.

## Small but intensely irritating.
Here is the small but intensely irritating part - *<strong>there is no easy way to do it.</strong>*. The best you can do is find the first one that matches a RegEx.

That's odd (_and I am being polite here_).

In Git, arguably one of the best softwares on the planet, *you can easily add a key-value pair*, but *you cannot easily retrieve it by key*.

What?

Let me rephrase this - You are using an excellent programming language. Probably the best ever. In that language there is a dictionary or hashtable or map implementation. It allowes you to stuff things into a key-value collection by a key (random string). But here's the catch, it does not give you way to retrieve the same by name. Not easily anyway.
- Now list down your opinion of the programmer, designer and architect of the person who did this. And then... 
- Map the attributes you wrote down against the names Linus Torvalds or Git. 

(The second bullet point above would probably result in something resembling that OCD nightmare meme - a leaning tower of Pisa in a square box in a square wall picture frame. IT. DOES. NOT. COMPUTE. How did these people miss this?)

## Software has issues. Period.
Ok, hyperbole apart, this is basically what software is. 

Take the best software, built by the best people - It doesn't matter. Something, somewhere is inevitably going to fall off the radar, or the planning, or the execution, or the design. 

In this case Git is probably the best DVCS out there, conceived by a marquee brand leader, built and maintained over many years by a great team, yet the fact that if you stored something by name, you would want to retrieve it by name seems to have escaped all those people who participated in it over all the years. 

And, as is usually the case, it never became important enough to get put back in. 

## So, Git Unstash.
Yeah, ok software got issues. But we got developers. And developers fix issues. <small>_And what's a technical blog post without a solution to an issue anyway?_</small> 

So here's how to do unstash in Git.<small> _(even though its available just a couple of google clicks away)._</small>

### Git - stash by name.

This works just so - 

```git stash save "thisName"``` 

The above is deprecated but still works. The current recommended way is - 

```git stash push "thisName"```.

### Git - unstash by name.

This is the iffy thing. you can do this - 

```git stash apply stash^{/thisNa}``` 

Notice the *'thisNa'*, that's a regex which contains part of your stash message. Additionally, what this does is retrieve the first named stash value which matches. Which means, if you do this 

```git stash save 'thisName1'``` 

and this 

```git stash save 'thisName2'```, 

You are out of luck during retrieval unless you can remember the exact second phrase, like this - 

```git stash apply stash^{/thisName2}"```, 

Since Git returns only the first match, using part of the phrase leads you to the first partial match which may or may not be what you want.

There are alternatives which involve using a combination of 

```git stash list```

and 

```git stash pop/apply stash@{n}```

where n is a number referring to nth stash from top, etc (_and use either pop or apply, not both_). 

For additional tips and sources and other more detailed knowhow, look up this [SO question](https://stackoverflow.com/questions/11269256/how-to-name-and-retrieve-a-stash-by-name-in-git).

## Bottomline.

Software, even the best software, has nuances and idiosyncrasies. Stop cribbing, drink your medicine, and get back to work(arounds).













