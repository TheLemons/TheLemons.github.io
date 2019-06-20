---
layout: post
title:  "RedBluePlayer - Status Update June 19, 2019"
date:   2019-06-19 22:55:50 -0400
categories: RedBluePlayer
---

A potentially more interesting (but drier) post this time. For the most part, there are only two categories of work that has occurred since the last post: data massaging/entry and optimizations made (and associated testing).

***

## Data Entry and Massaging

Last post, I made a note that I needed to alter the known sprites to reflect the color palette of the emulator. For most of the environment sprites, this isn't too bad. I've only observed four colors used across all sprites, so detecting four colors and converting them to the new ones is fairly trivial. Some of the oddities has come up when only three colors are detected. This has meant that the color codes for black and white translate well but there is a shade of gray that has become a guess. For some of the sprites, I've just analyzed the data in the bitmap files and taken a guess at what it translates too. If I'm feeling particularly lazy, I could just have an output of both shades of gray relative to the emulator color codes.

Additionally, I have done some work to associate available actions to environment sprites. These actions include: the ability to surf, the ability to walk onto the tile, if the the tile will warp the player, or if the player can do nothing at all with the tile. Naturally, this process hurt. I have over 800 environment sprites and I have to look at each and every one to associate an action with, most of which aren't obvious with the sprite alone what the image is (e.x. a portion of the machinery in the Cinnabar Laboratory). I know I will continue to alter these values (and add more sprites), so I will continue with other work while slowly making adjustments when necessary. More on this in the "Next Steps" portions.

*** 

## Actor Detection, Tries, and Speedups

Now, for the more interesting part. I have implemented a fuzzy match solution for actor sprites using a trie data structure and I have noticed a considerable speedup when analyzing the Kanto overworld map.

For those of you not familiar, a [trie]({https://en.wikipedia.org/wiki/Trie}) is a tree-like data structure. Trie branches/subtree represent portions of the data stored within, such as each character within a string from the Wikipedia link. For this particular implementation, I probe specific pixels within a proposed tile. Each node traversal represents a matched color. If the match does not have a subtree, the search is a miss. Rather than doing this until exhaustion, the implementation accepts a list of indices to probe and constructs the trie so that leaf nodes are at the same depth.

Each leaf node within the trie stores a vector of potential matches that are then compared against. I cannot use a trie to get a definitive match because actor/object sprites can appear on top of environment sprites, resulting in the background interfering with detecting the actor. In one-to-one comparisons I've done, I ignore white pixels since the image backgrounds I have are also white. I could recolor the sprites but that is undesirable at the moment as it greatly increases the amount of work I have to do just massaging data.

Now, with a optimal set of sprites, we can greatly reduce the number of sprites to compare against. Each branch traversal can optimally eliminate 3/4 of sprites for the environment and 2/3 for actors (actors appear to only have three colors each). As a heuristic solution, I probe 6 pixels for actors, all of which are center within the tile and appear to always belong to the actor sprite and not the environment background.

***

![](/assets/kanto-rgb-emul.png){:width="640px"}

The Kanto overworld map is ~1400 times larger than the regular images expected to process with the GameBoy screen. Much of it is a collection of white cells needed to pad out areas and should not match anything.

When processing matches of the overworld using a HashMap for the environment and linear, one-to-one comparisons for actors (the old way of processing), processing takes ~30 seconds to complete. When processing the overworld map for actors only using the old method, processing takes ~60 seconds to complete. This is because the environment detection occurs first and identified cells are skipped when detecting actors.

When processing matches of the overworld using the Trie implementation for both the environment and actors, processing takes ~25 seconds to complete. Identifying only actors within the overworld takes ~13 seconds, a significant decrease from 60 seconds.

However, the current method of identification going forward will be a mixed approach of using a HashMap for the environment sprites and a Trie for the actors. Processing the overworld takes ~20-22 seconds and appears to be the fastest at present. 

**

I am still not pleased that I am taking an entire Vec<u8> and using it as a key within a HashMap but it appears to be the fastest method still. I am currently using 6 probe points for the actor sprites and have tested the Trie for environment sprites using upwards 9 probe points. The actor probe points come from indices that should always belong to the actor sprite itself, making them safe places to test. The indices used for the environments came from comparing the color codes for all sprites I have and finding uniformly distributed indices (e.x. the 72th pixel appears to have 24%-27% chance for all colors, making it a good candidate to eliminate bad matches). 

I am constantly learning Rust and my implementation is definitely not satisfactory or ideal. However, I feel I should share what I have currently. At the bottom of this blog post should be the implementation. There are bad practices, such as magic numbers. However, I implemented this after fighting the Rust compiler for some time and just wanted a proof-of-concept. Things will be tidied up with time.

**

## Next Steps

My next step will be to add another display/window to the emulator and attempt to attach the processing logic. The goal will be to let my recognition functions do their work and display to the second screen using various color codes so I can visually debug potential issues. When combined with the screenshot implementation I shoehorned into my RBoy fork, edge cases should be resolved fairly quickly. As I've stated before, data massaging will most likely be an ongoing issue.






SpriteData struct 
```Rust
#[derive(Clone)]
pub struct SpriteData {
	pub data: Vec<u8>,			// The raw pixel data (not headers or anything)
	pub filename: String,		// File extracted, used for some settings and comparison
	pub action: SpriteActions,	// Player can do or expect something here (or Nothing)
}
```

```Rust
use spritedata::SpriteData;

enum TrieData {
    Subtrie(), // Not a leaf, the data and the index
    Leaf(Vec<SpriteData>), // Leaf node w/ data
}

struct TrieNode {
    data: TrieData,
    children: [Option<Box<TrieNode>>; 4],
}

// 0x00, 0x60, 0xc0, 0xff
impl TrieNode {
    fn new(d: TrieData) -> Self {
        TrieNode{data: d, children: [None, None, None, None]}
    }

    fn node_search(&self, data: &Vec<u8>, i: usize, actor: bool, indices: &[usize]) -> Option<SpriteData> {
        match self.data {
            TrieData::Leaf(ref matches) => {                for v in matches {
                    if actor && v.data_actor_equals(data) {
                        return Some(v.clone()); 
                    } else if !actor && v.data_equals(data) {
                        return Some(v.clone()); 
                    }
                }

                return None;
            },
            TrieData::Subtrie() => {
                if i >= indices.len() {panic!("Bad index value: {}", i);}
                let test = data[indices[i]  * 3];
                let next_index = match test {
                    0x00 => {0}, 0x60 => {1}, 
                    0xc0 => {2}, 0xff => {3}, 
                    _ => {return None;}
                };

                let node = &self.children[next_index];
                match node {
                    None => { return None;},
                    Some(ref n) => {return n.node_search(data, i+1, actor, indices);},
                }
                
            },  
        }
    }


    fn node_insert(&mut self, spdata: &SpriteData, i: usize, indices: &[usize]) -> bool {

        match self.data {
            TrieData::Leaf(ref mut matches) => {
                assert!(i == indices.len(), "Depth for Trie is off: len {} i {}", indices.len(), i);
                matches.push(spdata.clone());
                return true;
            },
            TrieData::Subtrie() => {
                assert!(i < indices.len(), "Index given incorrect for INDICES array. len {} i {}", indices.len(), i);
                let val = spdata.data[indices[i] * 3];
                let child_index = match val {
                    0x00 => {0}, 0x60 => {1}, 
                    0xc0 => {2}, 0xff => {3}, 
                    _ => {return false;}
                };

                if self.children[child_index].is_none() {
                    if i == indices.len() - 1 {
                       self.children[child_index] = Some(Box::new(TrieNode::new(TrieData::Leaf(Vec::new()))));
                    } else {
                        self.children[child_index] = Some(Box::new(TrieNode::new(TrieData::Subtrie())));
                    }
                }

                match self.children[child_index] {
                    None => {
                        // No, really. This should be set as above.
                        panic!("God help ye...");
                    },
                    Some(ref mut n) => {
                        return n.node_insert(spdata, i+1, indices);
                    },
                }
            },
        }
    }

}

pub struct Trie {
    root: Box<TrieNode>,
    indices: Vec<usize>,
}

impl Trie {
    pub fn new(i: &Vec<usize>) -> Self{
        Trie{root: Box::new(TrieNode::new(TrieData::Subtrie())), indices: i.clone() }
    }

    pub fn insert(&mut self, data: &SpriteData) -> bool {
        return self.root.node_insert(&data, 0, &self.indices);
    }

    pub fn search(&self, data: &Vec<u8>, actor: bool) -> Option<SpriteData> {
        return self.root.node_search(data, 0, actor, &self.indices);
    } 

}
```