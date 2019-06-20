---
layout: post
title:  "RedBluePlayer - Initial Image Analysis"
date:   2018-07-25 22:55:50 -0400
categories: RedBluePlayer
---

So, I've been slowly poking away at overworld screen recognition. As I stated in a previous post, the images appear to be accessible as 24-bit bitmaps within memory (obviously, without the headers). As a result, I've spent most of my time just taking screenshots, making sure I'm traversing the data properly, and collecting spritesheets to compare with. 

With a little Googling, info about the Gameboy screens can be found. The Gameboy screen is 160 pixels wide and 144 pixels wide. The bitmaps are 3 byte-colors, one for each color (RGB) and without an alpha channel. When you look at the in-game screenshots below (which will be explained in a bit), you can see 16x16 boxes in a 10x9 grid.

Because the output buffers are just bitmap data in memory, it's probably best to output and process screenshots as bitmap files. I wrote some C-code with various functions to assist me in both processing the files and understanding the file format. This is all local at the moment, but I may commit the spaghetti code, most of which is disabled at the moment. Expect nothing pretty from that.

To better see what squares are used for the environment, I tried to insert lines between each 16x16 square. I missed a few important pieces to the bitmap file format while inserting additional pixels, and decided the best course of action was a color-shift instead. 

![](/assets/test31.bmp){:width="320px"} ![](/assets/test31_shift.bmp){:width="320px"} 

![](/assets/test33.bmp){:width="320px"} ![](/assets/test33_shift.bmp){:width="320px"}

With little trouble, I managed to get each square colored individually. And, just as I had hoped, various pieces fell into place. The buildings are multiple grids in their entirety. Terrain corners occupied entire squares. Some areas seem to have some extra texturing, such as the corners of rock in Fuchsia City. However, this probably isn't a big issue.

The eagle-eyed of you may have noticed something a little off though. The player sprite doesn't quite fit inside of a grid.


![](/assets/test31_shift.bmp){:width="320px"} ![](/assets/test31_shift2.bmp){:width="320px"}

![](/assets/test33_shift.bmp){:width="320px"} ![](/assets/test33_shift2.bmp){:width="320px"}

![](/assets/test43_shift.bmp){:width="320px"} ![](/assets/test43_shift2.bmp){:width="320px"}

The offset is four pixels higher for objects and NPCs in the world, which means that the tops of the topmost NPC/object is clipped and the NPC/Object just offscreen to the south will not appear. If we truly care to recognize each and every pixel on the screen, we will probably have to do two sweeps of the buffer. There are other potential ways of doing this, but I think a function for each is probably the easiest and most reliable way at present.

From this point forward, not just for this post, I want to make a distinction for objects/sprites: environmental and objects. These aren't hard set, as environmental tiles may be interacted with or may affect the state of the game. However, this distinction exists to separate how these objects are identified.

***

I am currently processing a spritesheet I found online to recognize environment sprites. I altered my local C code to be a little more accepting of the bitmap files it reads, which probably should have been done to begin with. The terrible part now is associating each tile with a flag that identifies what can be done with that tile. In all the above screenshots, you can see: doors, immovable rocks, ramps, flooring, ledge, and even a boulder switch. All of these have distinct purposes for both gameplay and traversal. As a result, _sigh_ all must be entered with flags.

Data entry sucks. I'll check back in when I get the spine to do what needs to be done.
