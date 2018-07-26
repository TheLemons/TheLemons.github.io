---
layout: post
title:  "RedBluePlayer - Proposal"
date:   2018-07-25 19:55:50 -0400
categories: RedBluePlayer
---
So, a silly idea that I've had for years is to build a bot that plays pokemon. Not a ROM hack, not some fan-game project. I've wanted to make something that "sees" the screen (i.e. parses an emulator's output buffer) and makes decisions based on the state of the game. At the time of writing, I've been slowly working on things and I figured it was about time I started writing things down.

Now, before I get too far, nothing will probably come of this project. It obviously takes a great deal of work with a good deal of setup and prep before you start getting into any of the "interesting" material (e.x. pathfinding, battle strategy, etc). And, between life's distractions, digital distractions, and just a lack a focus, there's no telling when I will and will not work on this project. So, this blog is just as much to keep my thoughts straight as it is to *maybe* have someone learn from any ramblings I put out.

Something else to mention is that I don't plan on working on a solution based on regular expressions, as seen in this [talk]. Typical video game bots will operate with state machines. While these will operate as FSAs (only so many states can be implemented), I don't plan on hashing out a formal grammar for pokemon. Sorry =P 

### Work Done So Far

So far, I've started work processes sing bitmap images. What I have seen in the VisualBoyAdvance-M and CoffeeGB projects are bitmap, output buffers accessible with a little tinkering. This has come with a little extra prep work that won't go into the project prop, and I'll have a separate post for that. 

Currently, I have sprite data sitting in a large array of C-style structs waiting to be used. I want to process an overworld image I downloaded to build the start to a map. If I can process that, then I have the option to work on pathfinding, which will most likely just be an [A* variant].


### Work To Be Done

So, this will be hashed out in more detail in a different post, and will most likely be updated *very* frequently. So, at a rough level:

* Overworld recognition
* Basic Pathfinding
* Battle recognition
* Battle execution
* State transitioning between overworld and battles
* Complex pathfinding
* Waypoint/queue system

I will break these down further in a follow-up post (and this list may very well end up out of date as progress occurs).

In addition, I haven't solidified the emulator I want to use. My current picks are CoffeGB and VisualBoyAdvance-M. CoffeeGB is written in Java, and I would use Java or Scala to add my own code (I am currently learning Scala on the side, too). The downside is that it is fairly simpler and my current bitmap code is in C because I am an idiot. VisualBoyAdvance-M is a bit more tried and true, but comes with a good deal of extra code for both WxWidgets and just additional features.

[talk]: https://www.youtube.com/watch?v=Q2g9d29UIzk
[A* variant]: https://en.wikipedia.org/wiki/A*_search_algorithm