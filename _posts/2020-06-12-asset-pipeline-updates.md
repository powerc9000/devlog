---
title: Asset pipeline updates
date: 2020-06-12 20:32 -0600
---

[In a previous post](/devlog/2020/06/02/asset-pipeline.html) I talked about my asset pipeline.

I have now made a few changes/improvements to the system.

### SQLite

[SQLite](https://www.sqlite.org/index.html) is a pretty neat tool. It's a database system saved to a single file. I am using it to store information about my game assets.

In my previous pipeline I was adding every PDF to a simple text line with a bit more info. It was easy and worked, but I started to want a little more data about assets individually.

Even if certain assets are grouped together in a PDF I wanted to be able to add data to an asset on an individual level. Maybe different assets should have different scaling, or associated tags, or making animations easier.

All this can be done in a simple text file, but it's fairly relational data so why not use a simple database to store the info.

For my tables I made a few

* raw_asset, the PDF file for an asset.
* game_asset, the individual assets as used in the game.
* asset_tag, tags for individual assets.
* animation, animation info.

It's pretty simple. raw_assets speficy a name for the group of assets in it and the path to the PDF. 
Game assets have a raw_asset they belong to that specifies where in the PDF they are by path eg. `Group/layer name`.
Then game assets can have tags and what animation they belong to.
Animations are just an id and a name.

### Effects

These changes have made the pipeline a bit simpler in some ways and added a bit more complexity to other things, but at the benefit of providing more robust information.

We no longer need to do much inspection of the PDF to get groups and layers in ruby. We can just query the database about the raw assets and for each raw asset get its associated game assets.
Since the game asset tells us in the raw asset where it is, we can just pass that info straight to gimp to export the PNG.

Before the name and path of the asset in the sprites folder gave us metadata about it. `Harper/Front/walk_1.png` said the group name and the facing direction and the animation and the frame.
While giving some information it's hard to add more. 
With the database existing the script doesn't need to save to paths with names that encode data about our asset. The script just saves by the id from sqlite and the group name e.g `Harper/26.png` (using the group name as a folder purely to make them easier to see in the finder). 

When it comes to packing the texture we have all we need to know: the asset id and where it is in the texture atlas. Outputting a much slimmer file that's is pure JSON from texturepacker. It also removes any weirdness with figuring out the frame number from the asset name in the custom formatter in texture packer.
```

{
	"frames": [
		
			{
				"filename":"battle/28",
				"frame":{"x":1,"y":3478,"w":500,"h":216},
				"trimmed":true
			}, 
			
			{
				"filename":"buildings/20",
				"frame":{"x":497,"y":1709,"w":242,"h":500},
				"trimmed":true
			}, 
			
			{
				"filename":"buildings/21",
				"frame":{"x":1,"y":564,"w":500,"h":412},
				"trimmed":true
			}
	],
	"meta": {
		"image":"sprites.png"
	}
}
```
I made a script with Node.js that takes the texture packer json file. Looks up the asset in the db and adds any other metadata we care about to it. Such as the frame number, the tags, the id and group.
It still formats it as lua because I didn't want to change my engine to read json right now...
```
return {
	frames = {
		{
			filename="battle/28",
			frame={
				x=1,
				y=3478,
				w=500,
				h=216
			},
			trimmed=true,
			tags={
				"battle","banner"
			},
			animationName="",
			id=28,
			name="battle_banner",
			group="battle",
			frameNumber=-1
		},{
			filename="buildings/20",
			frame={
				x=497,
				y=1709,
				w=242,
				h=500
			},
			trimmed=true,
			tags={
				"regular"
			},
			animationName="",
			id=20,
			name="some_building",
			group="buildings",
			frameNumber=-1
		}
	}
}
```

Since SQLite can give us always unique ids that are consistent across runs for my assets, it makes it easy to build tables about animations in the engine. 

Previously when I loaded this data in my engine I was building up animation info and asset info in the same loops. It felt fragile and hacky. The engine now just loads all my assets info from the lua file then builds up animation info as a seperate step.

After the engine loads all the asset info it has a consistent id for an asset that it can always reference in my game and ask the database questions about it if I need it to. (which I do!)

With all the asset info being loaded the engine can build up the animation info. In my engine I just make a call with `popen` and run `sqlite3 database.db "select * from animations"`.

This will produce a list containing an id and name for all animations. For each animation I can now query about which asset ids are assocated eg `sqlite3 database.db "select id from game_asset where animation_id=2"`.

Now the engine will make up some animation info tables. Essentially a map of animation names and an array of assets ids that are part of the animation. 

Later in the gameplay code if the character is walking I can tell the engine just to use the animation named`"harper_front_walk"`.

Pretty fun!

Anway here's a picture of a new things I drew Grambly

![Grambly is a weird looking red skeleton. He loves God and hates cops. Amen.](/devlog/images/grambly.png)







