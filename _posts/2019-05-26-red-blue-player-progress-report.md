---
layout: post
title:  "RedBluePlayer - Actor Recognition and Emulator Updates"
date:   2019-05-26 22:55:50 -0400
categories: RedBluePlayer
---

More work related things to be done, so not as much progress as I would like. Let's get right into it.

***

## Actor/Object Recognition

When I left off last time, I had basic environment detection working. The green tiles in the previous post were pieces of the environment that were successfully recognized. The next logical step would be to get actors and objects recognized, which I have.

![](/assets/test_color_ident_54.bmp){:width="320px"} 

Similar logic applies when compared to the empty environment identification with two exceptions. First, actors/objects sit on top of the environment, so comparisons must be done against the known sprites while ignoring any background. At the moment, the known images have a white background, so the white pixels are ignored while black and gray pixels are validated. Second, the top row is clipped by 4 pixels. As a result, only a partial match is used.

## RBoy Tinkering and Update

I've had my eye on RBoy for a while. I currently have a fork and I have made some edits that I have yet to commit to my repository. I made two changes. First, the constant used for timing the clock was way too slow for me. There is a magic constant within the code that I've updated to be bit more reasonable for me. The downside is that I still have a magic number and the audio is mismatched. This will require some additional work.

Second, I've added the ability to take screenshots. The code isn't that pretty and is mostly copy-pasted from the bitmap output code I had previously. The currently process is that the button press event copies the screen data and sends it to an idle thread via a channel. However, once I started collecting screenshots, I noticed something was different: the color pallets were different from the static files I had previously.

## Data Massaging

So, the matching I had previously was quite loose. White `0xff` could be matched with `0xff` or `0xf8`. Same with other grays and I do not recall seeing black as anything other than `0x00`. To get a bit closer to what is expected, I had to massage the known sprites and make the colors present one-to-one with what the emulator will present. Most of the time, it was fairly easy. Things like getting a sorted list of the four colors present, map it to the emulator colors, and output to a new file. A few problem cases have presented themselves, like images with only 3 colors present. I believe I have most of the known sprites present but I know I will be massaging values for some time.

On the left: the previous file with the old color codes. On the right: the current RBoy-compatable file.


![](/assets/test47_orig.bmp){:width="320px"} 

![](/assets/test47_emu.bmp){:width="320px"} 

## Optimization Attempts

I changed how environment sprites are detected in the raw bitmap data. Rather than comparing each pixel one-by-one across all the known environment sprites, I've changed to a HashMap implementation. The key in the HashMap currently is the `Vec<u8>` of the sprite. I am not fond of the key at the moment and I currently know that what I have does not contain a collision with the DefaultHasher with Rust. I may try to rework this as having a 768 byte key is off-putting but hashing a hash is odd to me and I'm not sure what else I want to try. I ultimately want to eliminate the guess work of finding a match and want something faster, whether that be a hash-based solution such as this or something like a trie where examining the input leads to the solution without "guessing".

I do not have benchmarks at the moment. Processing the Kanto map appears to be significantly faster than before, which is a victory in of itself. The actor identification takes significantly longer, even with some logic that flags tiles as identified already. Unfortunately, I can't think of a way to speed up the actor identification as the background bleeds in.

***

I realize that this is a late and weak post. Most of this work has been done for some time, save for the optimization attempts. Next will be some more massaging and, hopefully, the start of A* path finding. 
