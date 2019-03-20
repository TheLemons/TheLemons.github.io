---
layout: post
title:  "RedBluePlayer - Status Update March 20, 2019"
date:   2019-03-20 18:29:50 -0400
categories: RedBluePlayer
---

Wow, it's been some time. 

Admittedly, I stopped working on the project after a few factors were taken into account. First, my day job took over a little bit and I spent portions of my free time looking into other technologies and languages. Second, I became a little discouraged when I discovered another pokemon bot existed.

I discovered at one point [Pokebot]({https://github.com/alexkara15/PokeBot}). For some reason, I thought my project would be novel and somewhat unique, the magic of which was ruined somewhat later. However, after looking at the Pokebot source code, it seems that the means and end goals of these projects are unique, meaning that there are still things to be learned and shared.

***

Now, while tinkering on my end, I decided to rewrite what I had in Rust. I wasn't looking for safe code and I didn't rewrite things because I was using C prior. I merely wanted to learn Rust and I decided to go forward with it. If I do not decide to change my arbitrarily again, I will probably use [RBoy]({https://github.com/mvdnes/rboy}) as an emulator, which would need to be forked and used.

After some tinkering and learning on my part, what I had prior works as expected within Rust and I even corrected an issue due to some laziness on my part prior. Rather than moving forward with data entry, I decided to try and get image recognition working. The image below demonstrates what was recognized with green regions.

![](/assets/test_color_ident_23.png){:width="320px"} 

The green areas are environmental only because I do not have a supply of object/NPC/player sprites yet. The regions without green are mostly due to the NPC/objects not aligning perfectly, hence why there are always two under an NPC or object. I also still need to add a couple sprites for flowers and water as they change, however I have some test code that can extract those for me to make saving painless.

I do still have bugs to work out. I have a Kanto map taken from online that I ran the sample code against below. 

![](/assets/kanto_color_ident.png){:width="640px"}

The NPC and object mismatches are to be expected, as are some of the water and flowers. What needs further work is the missing matches of environmental sprites. I do not have any proof yet as to why they do not match but I am assuming my math used for indexing into the bitmap is off at the moment.

***

I currently have the time and motivation to work on this again. I'll try to get sprites to use that will allow me to identify NPCs and objects within the images. After that, I will attempt to tag sprites with attributes so I can move onto basic pathfinding.
