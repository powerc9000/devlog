---
title: Asset Pipeline in my Game
author: Clay Murray
---

Every game has assets, and somehow those assets have to get into the game. Here's what I do.

## 1. Creating the Assets

So first step is obviously creating the assets. What I am doing is different from most people. I draw the assets in a sketchbook, then I scan those images into my computer.
After that I take the scans and mask out all the parts I don't want, rotate stuff, and make everything feel real good.

### Photoshop organization

When making an asset for some entities you want animations to have animations on. And one of my goals is keeping the same character's asset data in the same PSD.

![PSD Layout example](/devlog/images/psd_layout.png)

So in my PSDs I just groups. With the name of the group being a direction (you can go 4 cardinal directions in my game) and a frame or still image in that direction. Animations cycles are specified via a name_framenumber syntax (eg walk_1)

For other types of entities like buildings I don't really care to have animation frames. In those cases I don't use groups or anything.

## 2. Exporting the Assets

Now that the assets have been created we want to export them. You could have a script that just looks for PSD files in a directory and exports them all. However, I want a bit more granularity in my process.
I am using a really simple file format to specify the assets I want to use and some other information about them.

```
assets/raw/harper.psd harper group 75%
assets/raw/some_building.psd buildings single x500
assets/raw/the_store.psd buildings single 500x
assets/raw/robo_dude.psd robots single x400
assets/raw/pointy_boy.psd ui single x50
assets/raw/square_boy.psd robots single x400
assets/raw/roller_boy.psd robots single x400
assets/raw/skelly_boy_blue.psd skeletons single x400
assets/raw/dewey_cheetum_howe.psd buildings single 500x
assets/raw/battle_banner.psd battle single 500x
```

The format is just space seperated items in a simple text file. 

* Path to PSD
* Grouping to save to
* If this PSD uses groups or if it just has a single layer.
* imagemagick scale factor.

I wrote a ruby script that takes this info and reads the lines.

Over each line it loads the resulting PSD, finds all the layers we want to export and prepares the directories.

I do a check to see if the PSD is newer than the result PNG or the result PNG doesnt exist. That way we only have to run the export on assets that have changed.

If it's a grouped PSD each group is a sub folder inside the group name and the layer name is the exported png name (eg harper/Back/walk_2.png).

If it's a single layer we save it to the group folder with the PSD file name as the resulting png name (eg buildings/the_store.png).

### Saving to PNG
The ruby PSD library I am using is too slow on its own to save PSDs at a reasonable rate. At first I tried using imagemagick but it doesnt accept layer names, only the scene index. Like: `convert file.psd[1] out.png`

I could never find a way to map between a imagemagick scene and a layer in my ruby PSD stuff but Imagemagick is fine for single layer PSDs and I use it to export those.

#### GIMP

Well as it turns out gimp has a scripting interface that doesn't even have to show a GUI. I figured out how to use it to export my grouped PSD files. 
Here's the process

1. Ruby passes the group and layer name to gimp along with the path to the PSD and a path to save the final image to.
2. Gimp finds that layer, makes sure it is visible and saves it to the path we gave.
	* this process is somewhat weird. If you just try to save the layer and only the layer it won't include the clipping mask. So you need to make the layer you want visible and hide all the other layers. Then you can merge visible layers and save the result of that. 

Here's what my resulting GIMP script looks like:

```
#!/bin/bash
set -e

# Start gimp with python-fu batch-interpreter
/Applications/GIMP-2.10.app/Contents/MacOS/gimp -i --batch-interpreter=python-fu-eval -b - << EOF
import gimpfu
import traceback

def convert(filename, groupName, layerName, out):
	try:
            print(filename, groupName, layerName, out)
            img = pdb.gimp_file_load(filename, filename)
            if len(groupName) == 0:
                layer = pdb.get_layer_by_name(layerName)
                pdb.gimp_file_save(img, layer, out, out)
            else:
                for layer in img.layers:
                    if pdb.gimp_item_get_name(layer) == groupName:
                        children = pdb.gimp_item_get_children(layer)[1]
                        for child in children:
                            item = gimp.Item.from_id(child)
                            if pdb.gimp_item_get_name(item).split(" ")[0] == layerName:
                                pdb.gimp_item_set_visible(item, True)
                            else:
                                pdb.gimp_item_set_visible(item, False)
                    else:
                        pdb.gimp_item_set_visible(layer, False)

                nLayer = pdb.gimp_image_merge_visible_layers(img, CLIP_TO_IMAGE)
                pdb.gimp_file_save(img, nLayer, out, out)
            pdb.gimp_image_delete(img)
            pdb.gimp_quit(0)
	except RuntimeError as err:
                print(traceback.print_exc())
                pdb.gimp_quit(1)
		
convert('$1', '$2', '$3', '$4')
EOF
```

(at one point I was going to use gimp for the group and single layer path so the logic is still there)

#### Back to ruby

After gimp has our file saved or image magick exported our layer. We write an entry to another txt file this is a list of files that we want imagemagick to resize and the size to make it.

It ends up something like this:

```
assets/sprites/harper/back/walk_2.png 75%
```

#### Imagemagick

With a simple bash script we take each line of the file and resize it according to the sizing info in that line. 

It can be passed whatever sizing info imagemagick can accept. Percentages, WIDTHxHEIGHT, xHEIGHT, WIDTHx etc.

Now we have all our images exported to folders.

### Texture atlas

Now I want the images packed into one texture atlas PNG. I use TexturePacker Pro. <small>I paid for pro so I could make a custom exporter.</small>

TexturePacker can export from the command line making it easy to include in the automated pipeline.

All texturepacker is doing is looking for files in the sprites directory and adding them to the texture atlas.

TexturePacker exports to a lua file that the engine will read.

## Into the Game

Now we have all our stuff exported into a texture atlas with a lua file describing the bounds and names of all the files. I embedded lua in the engine and we read the file and get the resulting data from it into odin. At some point I could reload assets on the fly. I just don't do that yet.


The cool part of all this for me is having a fully automated pipeline. No exporting from photoshop or opening other tools. Just works from one command.

Video of it running.
<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/ObecwEU0QGc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>




