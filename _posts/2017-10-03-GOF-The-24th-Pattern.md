--- 
layout: post
title: "GOF - The hidden pattern."
author: "Author"
comments: true
tags : [Pattern, Analysis, Strategy, Design]
---

The Gang-Of-Four book has 24 patterns. And it only took me only about 12 years or so to get it. Considering that I first encountered the book in 2005.

I know, I know. I can hear the collective tirade of protest. All the analysis and the literature says 23. 

Anyone who can count to 23 says 23. 

The number of pattern descriptions in the book are 23.

In this case at least, as far as common knowledge goes, 23 is the answer, not 42.

Yet I will argue. There is another pattern in there. A pattern that is repeated over and over. A pattern that embeds yourself in your psyche and you use it unconsciously for several years until it hits you. 

You disagree, eh?  Ok then, we'll try a different tack. 

## Pattern No. 24. An oblique introduction.
Are you familiar with the word 'concept'? *Yes. Good. Thank you. A no here would have made it really difficult to proceed*. 

So you have maybe a concept or two of your own. *Yes, Even better*. 

Now here's the kicker - explain your concept to me, in writing. (*It doesn't have to be technical even. It could be in woodworking for all I care*). But try writing down the concept. 

Yes, go right ahead. Get some notes, stickers, postits, onenotes/evernotes, mindmaps and just write out the outline of your explanation. I can wait. See I have coffee, blogs, white-noise. I am all good with waiting a bit.

Writing down a concept is not an uncommon activity in the software engineering line of business. We are asked to explain concepts all of the while, in writing - RFP submissions, software design documents, new initiative proposals, POC summaries. I have done my fair share.

Recently I did one more - A *'starting devOps'* analysis. And it was while doing this that I also started to think about '*how to explain*' such a thing to someone else. And I found that everytime I need to explain a concept, I tend to use the same patterns to do so. It was a curious co-incidence that design pattern fever was running rather high at the workplace at that period, and recurring discussions were happening around this famous book and its contents.

## Explaining yourself, preferably clearly.
So a quick look at what I do when writing down a concept? 

### Step 1
This part defines the motivations and reasons for the concept. In my case, It has traditionally involved consideration and listing down of the following
-   The driving forces
-   The constraining forces
-   The goals

### Step 2
Given the *drivers, constraints and goals*, I now proceed to 
-   An explanation of the general structure of the concept. 

### Step 3
Given the *drivers, and the structure*, we now drill down further into 
-   Key stake-holders 
-   Key components involved in the concept, 
-   Key interactions between components themselves or with a stakeholder, all being traceable to one or more of the motivations.

### Step 4
The next step is usually to play devils advocate. As such, we make a listing of 
-   Assumptions 
-   Tradeoffs
-   Areas of applicability

### Step 5
We are now reaching the close of the explanation. At this point, we should be citing
- References
- Documentation for further details.

### Step 6
No concept paper is complete without a brief note about cases which have a high chance of rendering the current concept invalid and leads us to look for alternative solutions. So we note down the 
-   Limitations of concept
-   Alternative possibilities.

Hopefully still here with me. And not too many objections. Ok. 

Now I will copy a passage from a textbook. The following section is a verbatim copy-paste from the GOF book

    Describing Design Patterns

    How do we describe design patterns? Graphical notations, while important and useful, aren't sufficient. They simply capture the end product of the design process as relationships between classes and objects. To reuse the design, we must also record the decisions, alternatives, and trade-offs that led to it. Concrete examples are important too, because they help you see the design in action.

    We describe design patterns using a consistent format. Each pattern is divided into sections according to the following template. The template lends a uniform structure to the information, making design patterns easier to learn, compare, and use.

    Pattern Name and Classification
    The pattern's name conveys the essence of the pattern succinctly. A good name is vital, because it will become part of your design vocabulary. The pattern's classification reflects the scheme we introduce in Section 1.5.

    Intent
    A short statement that answers the following questions: What does the design pattern do? What is its rationale and intent? What particular design issue or problem does it address?
    
    Also Known As
    Other well-known names for the pattern, if any.
    
    Motivation
    A scenario that illustrates a design problem and how the class and object structures in the pattern solve the problem. The scenario will help you understand the more abstract description of the pattern that follows.
    
    Applicability
    What are the situations in which the design pattern can be applied? What are examples of poor designs that the pattern can address? How can you recognize these situations?
    
    Structure
    A graphical representation of the classes in the pattern using a notation based on the Object Modeling Technique (OMT) [RBP+91]. We also use interaction diagrams [JCJO92, Boo94] to illustrate sequences of requests and collaborations between objects. Appendix B describes these notations in detail.
    
    Participants
    The classes and/or objects participating in the design pattern and their responsibilities.
    
    Collaborations
    How the participants collaborate to carry out their responsibilities.
    
    Consequences
    How does the pattern support its objectives? What are the trade-offs and results of using the pattern? What aspect of system structure does it let you vary independently?
    
    Implementation
    What pitfalls, hints, or techniques should you be aware of when implementing the pattern? Are there language-specific issues?
    
    Sample Code
    Code fragments that illustrate how you might implement the pattern in C++ or Smalltalk.
    
    Known Uses
    Examples of the pattern found in real systems. We include at least two examples from different domains.
    
    Related Patterns
    What design patterns are closely related to this one? What are the important differences? With which other patterns should this one be used?
    
    The appendices provide background information that will help you understand the patterns and the discussions surrounding them. Appendix A is a glossary of terminology we use. We've already mentioned Appendix B, which presents the various notations. We'll also describe aspects of the notations as we introduce them in the upcoming discussions. Finally, Appendix C contains source code for the foundation classes we use in code samples.

## The hidden pattern, finally

And here we are. The approach taken to describe the Patterns in the first place. 

It is repeated 23 times in the book. In each chapter, to be precise. If you read the book thoroughly, not only do you understand specific patterns in full, but you learn how to analyse and understand patterns. It like not just learning algorithms, but analysing them as well.

Ladies and Gentlemen, Geeks and Nerds, I give you the GOF Pattern No. 24.

## The 'Concept Elucidation Pattern'.

All yours, now.

-----------------------------------------------------------------------------------------------