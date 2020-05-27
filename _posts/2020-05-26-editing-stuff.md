---
author: Clay Murray
---

## Game Maps

So the maps of this game I just have in a lua file. I did lua because I thought embedding lua would be fun. It was fun. However, I decided I didn't really want to make a scripting interface as I am the only person on this project and hot reloading odin code has been good enough for me. But lua still has staying around in a couple of tools and is still embedded in the engine. I should really write an entire blog about all that. It functions pretty similarly to how JSON files would at this point. But I suppose I can do expressions and other fun stuff that I couldn't in JSON. 

### Problems

Just directly edting the lua file, even if I auto reload it on change, is not very visual and since I am getting to the point of wanting to implement more I decided I do need more visual stuff. 
I had tried to do some editing stuff a while back with Tiled but I didnt like having an external tool that didnt even match what I wanted to do. I also began to try to have an in game editor but I got discouraged. Likely because I had a lot more things to still implement in the engine to make it feel right.

### Solutions, maybe

However today I gave it another go and I feel a lot more excited about the whole situation. Excited enough that I just wanted to make a quick post to show off that I can drag things around in edit mode. No save or undo or other tools I need. But this does make me feel like I can actually do this easier than the last time I tried.

Here's a video:

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/mdzc_zeM5Mk" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

[Previous Post]({{site.baseurl}}{{page.previous.url}})
