---
layout: post
title:  "RedBluePlayer - Roadmap"
date:   2018-07-25 20:47:50 -0400
categories: RedBluePlayer
---

The following is a basic roadmap for the RedBluePlayer. Nothing fancy, and will probably be wrong. Prepare for frequent updates.


[ ] **Overworld recognition**

Just as it says. Recognize objects and terrain in the overworld. 

[ ] **Pathfinding**

Implement some pathfinding algorithm; most likely an A* variant. Have the bot navigate from one point to another without pokemon or trainer battles. Also relies on one or more areas saved and loaded during execution.

[ ] **Battle recognition**

Parsing the battle screen to recognize pokemon sprites, text, health, and afflictions. Should be simpler than the overworld processing. There are fewer areas on this screen to process.

[ ] **Battle logic**

Just as it says. Will probably be activated on a per-battle basis at first, meaning the bot will need to cycle through held pokemon to audit what's available.

[ ] **Completed Map**

For basic pathfinding, one or more maps will be needed. However, they may not be connected to connected areas, such as going from a town to the inside of a gym.

[ ] **State transitioning**

Basics when going from battle to overworld and back.

[ ] **Make sure loss-of-movement is down** 

Warp pads, sliders, etc should be tested and implemented at this point, even if the bot just sleeps

[ ] **Basic Waypoint/Queue system**

Just the basics for now. QUeue up work items with a weighted queue, maybe even if it's just to do wild pokemon battles, heal, return.

[ ] **Implement full queue**