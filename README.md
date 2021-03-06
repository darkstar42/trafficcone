Traffic Cone is a 2d and 2.5d (isometric) tile based game engine written for html5. It makes complex animations of sprites and tiles based worlds fairly simple. 
===

Traffic Cone is currently client side only, so for multiplayer games you'll have to provide your own server. But expect to see more on this front soon, we're actively working on Traffic Cone server.

Features: 

* Easily configurable sprite animations
* Simple to set up tile based worlds
* Mock-3D math for layering, depth and positioning is handled for you via Traffic Cone's world models.
* Basic Isometric pathfinding, AI and collision detection
* Custom draw routines on sprites supported.
* Granular control of sprite sequencing supported for more dramatic animations

===
Examples:
### Running the samples
This document is still a bit immature, but Traffic Cone comes with a set of sample that should help to illustrate some of the concepts below. They're webserver agnostic, but most of our testing has been with Nodejs using the Express framework. To run the sample, you essentially want to modify  public/tccore/NotConstants.js (because they're not) and change your DOMAIN_PREFIX value to your webserver of choice and place everything under public so that it is available via that path. If you have nodejs installed and want to get up and running quickly you can modify DOMAIN_PREFIX to http://localhost:8089/ and then install expressjs and then run node webserver.js from the root trafficcone directory. From there all examples will be available via http://localhost:8089/examples/FILENAMEHERE.html

### Create a simple 2D world with 2D sprites
The supporting example for this section can be found in public/examples/2D/2D.html. This example rips some art assets from Street Fighter 3 and shows you how to wire them together easily with some basic collision detection.

First and foremost, Traffic Cone requires two html 5 canvases to work with. To create a basic engine, you simply create a new instance of the Engine class and supply the canvas elements ala:

```js
    ga = new Engine(document.getElementById("gamescreen"), document.getElementById("backbuffer2"));
```
The canvas is where all of our animations will be drawn. Animations come in the form of Sprites. Sprites are essentially animations which each image in the animation is made of Frames. Depending on how uniform your resources are, there are a few different ways to create a sprite. The assumption behind a sprite is that you are using a sprite sheet, and hence have an image of an animation sequence of some kind. For example see: https://github.com/j03m/trafficcone/blob/master/public/assets/ryu/RYU_PUNCH1.png 

When we define a sprite, we essentially supply the definitions for this animation. Here is the mechanism that gives you the most control, whereby we define each frame of the movement. 

```js
//create a sprite
var ryu = new Sprite("ryu");
var ryu.setup(ga);

//tell traffic cone what the dimensions
var attack = [];
attack.push(new Frame(0, 0, 105, 127, fastAction));
attack.push(new Frame(0, 1, 105, 127, fastAction));
attack.push(new Frame(0, 2, 105, 127, fastAction));
attack.push(new Frame(1, 0, 105, 127, fastAction));
attack.push(new Frame(1, 1, 105, 127, fastAction));
attack.push(new Frame(1, 2, 105, 127, fastAction));
attack.push(new Frame(2, 0, 105, 127, fastAction));
attack.push(new Frame(2, 1, 105, 127, fastAction));

ryu.defineSequence("attack", "../../assets/ryu/RYU_PUNCH1.png", attack, 1);
```
Of course, that's fine if our sprite sheet isn't uniform is terms of spacing and positioning of each frame. If it is, we don't need to be so granular:

```js
    ryu.easyDefineSequence("fall", "../../assets/ryu/RYU_FALL.png", 5, 5, 120, 178, fastAction, 1);
```
Here after the name and image, we simply supply the number of rows and columns. Then the height and width of each frame, the speed at which each frame should be played and the number of times it should played. While we can't control the speed of each frame granularly, we have also had to write much less code.

To tell our game engine about a sprite, we make it aware with:

```
ga.defineSprite(ryu);
```

We can also define a special sort of sprite to be one solid background image. Now this sort of thing won't be applicable to our isometric games usually (see below for more details) but for games with a solid unmoving background this works:

```js
var bdImage = [];
bdImage.push(new Frame(0, 0, 512, 860, 0));
backDrop.defineSequence("backdrop", "../../assets/londonsubway/londonSubway.png", bdImage, playInfinite);
ga.setBackDrop(backDrop);
```
What I did here is I create a sprite with 1 frame and then told the game engine to use it as a backdrop.

### Creating an isometric game world with mock-3D sprites
The supporting code for this section can be found public/examples/Isometric.html

Creating an isometric world is similar to 2D. But, now instead of just throwing sprites onto a canvas, we're going to define a GameWorld, which in turn will give us access to a GameWorldModel which we can use to place sprites in various aspects of "world space" and have that translate seamlessly to "canvas space". To create an isometric game world:

```js
ga = new Engine(document.getElementById("gamescreen"), document.getElementById("backbuffer2"));
var gameWorld = new GameWorld(250, 250, 73, 73, GAME_WORLD_STYLE_ISOMETRIC);
```

After this you can populate the cells of your game world with various sprites. You can also tell the gameWorld what sort of rendering style these sprites have and if they are open. For example GAME_WORLD_CELL_UNDERLAY means the cell is always rendered under character aka "the floor" whereas GAME_WORLD_CELL_OVERLAY means the cell will appear over characters like a pillar or ceiling. In addition, cells can be available for characters to move into aka GAME_WORLD_CELL_OPEN vs closed off via GAME_WORLD_CELL_BLOCK. Let's look at the code from Helpers.js where we have a function MakeIsoMetricWorld().

First, let's make the sprites we need. We use one sprite sheet to represent all of the floor "tiles". See Cells.js for details. Then, we create a sprite that will look like a column. 

```js
worldCells = altarCells_Sprite(ga);
worldCells.setup(ga);
column = columnCell_Sprite(ga, "column");
```

Let's loop through the world and set each tile to a random floor tile from our sprite sheet:

```js
for (var i = 0; i < worldSize; i++) {
    for (var ii = 0; ii < worldSize; ii++) {
        //set a cell using the worldCells sprite, randomly select one of the 25 frames in this sprite.
        gameWorld.Cells[i][ii] = new Cell(worldCells, getRand(5), GAME_WORLD_CELL_UNDERLAY, GAME_WORLD_CELL_OPEN, i, ii); 
    }
}

```

Nice, but a completely open world with no obstacles is hardly any fun. Let's add some columns to random cells:

```js
for (var i = 0; i < columns; i++) {
	var xx = getRand(worldSize);
	var yy = getRand(worldSize);
    gameWorld.Cells[xx][yy] = new Cell(columnCell_Sprite(ga, "column" + i),0, GAME_WORLD_CELL_OVERLAY, GAME_WORLD_CELL_BLOCK, xx, yy);
}
```

Last, we need to tell our Engine about the gameworld so it knows what to render, and where to set the camera.

```js
ga.setWorld(gameWorld);
ga.setCamera(worldSize/2, worldSize/2);
```

Once we have a world set up we can add some isometric sprites to the mix. For example from many of our examples we use our demo zombie isometric sprite:

```js
var zom = zombie_Sprite(ga, "zombie1");
```

This zombie function wraps a call to our SpriteInventory class which is an abstraction around some of the sprite creation methods we reviewed earlier. It is important because it demonstrates how an isometric sprite differs from a 2d sprite. For an isometric sprite, we don't just "flip" a sprite when it changes direction as we saw in our 2d demo. In the 2D demo, each sprite had a series of animation representing a given state. When we wanted to change the sprites direction, we simply inverted the sprite.

However, for our isometric demos, each direction a sprite supports can have it's own animation. So, in this case create a hierarchy of sorts:

	State - a set of animations representings the state of the sprite, ie the action the sprite is currently taking.
		Direction - an animation for a given state, in a given direction
			Frame - a frame of a given direction animation.
			
We accomplish this by supplying additional parameters to the defineSequence function:

```js
 sprite.defineSequence(name, imagePath + imageName, sequence, state, undefined, undefined, direction);
```

Where the "direction" parameter indicates that a given sequence should be attached to a given direction. Directions are specified via:

```js 
var SPRITE_DIRECTION_UNDEFINED = "0";
var SPRITE_DIRECTION_NORTH = "-1";
var SPRITE_DIRECTION_NORTH_EAST = "-2";
var SPRITE_DIRECTION_EAST = "-3";
var SPRITE_DIRECTION_SOUTH_EAST = "-4";
var SPRITE_DIRECTION_SOUTH = "-5";
var SPRITE_DIRECTION_SOUTH_WEST = "-6";
var SPRITE_DIRECTION_WEST = "-7";
var SPRITE_DIRECTION_NORTH_WEST = "-8";
```

All 2D animations internally leverage SPRITE_DIRECTION_UNDEFINED by default. Most of the complexity outlined here can be avoided by creating a json sprite definition and then invoking the global TCSpriteFactory method as demonstrated in our sample zombie sprite file public/assets/zombie/zombie.js. Here we can see:

```js
function zombie_Sprite(ga, name, loadCallBack) {
    var template = TCSpriteInventory["de558a04-b5df-4af4-b196-4393d732bb84"];
    var zombie = TCSpriteFactory(template, name, ga, loadCallBack);           
    return zombie;
}
```

Here we can see grab a template def from TCSpriteInventory using a guid and then pass that template to TCSpriteFactory generating our zombie sprite. Previous to this call we set up a json definition which defines the sprite:

```js
//zombie
TCSpriteInventory["de558a04-b5df-4af4-b196-4393d732bb84"] = {
    usage: USAGE_NPC,
    prefix: "zombie",
    speed: 3, //todo: Sprites should have attributes like strength, speed, weight etc
    imagePath: DOMAIN_PREFIX + "assets/zombie/",
    audioPath: DOMAIN_PREFIX + "assets/zombie/",
    behavior:
	{
	    seek: "hero",
	    foundState: "attacking",
	    moveState: "walking",
	    leftGap: 10,
	    topGap: 10,
	    moveInterval: 100,
	    definition: DOMAIN_PREFIX + "tccore/behaviors/SpriteSeeker.js",
	    constructor: "SpriteSeeker(behavior.moveState, behavior.foundState)",
	    eventType: "NPC",
	    startState: "normal"
	},
    states:
	{
	    normal:
		{
		    name: "normal",
		    columnRange: { start: 0, end: 3 },
		    playState: -1,
		    sound: "neutral1.mp3",
		    speed: 200
		},
	    walking:
		{
		    name: "walking",
		    columnRange: { start: 4, end: 11 },
		    playState: -1,
		    sound: "neutral6.mp3",
		    speed: 200
		},
	    attacking:
		{
		    name: "attacking",
		    columnRange: { start: 12, end: 21 },
		    playState: 1,
		    sound: "attack6.mp3",
		    speed: 100
		},
	    hit:
		{
		    name: "hit",
		    columnRange: { start: 21, end: 24 },
		    playState: 1,
		    sound: "gethit1.mp3",
		    speed: 300
		},
	    headsplode:
		{
		    name: "headsplode",
		    sound: "death2.mp3",
		    columnRange: { start: 30, end: 37 },
		    playState: 1,
		    speed: 300
		},
	    dead:
		{
		    name: "dead",
		    sound: "death2.mp3",
		    columnRange: { start: 24, end: 29 },
		    playState: 1,
		    speed: 300
		}
	},
    neutralState: "normal",
    rowDirectionMap:
	[
		{ row: 0, direction: SPRITE_DIRECTION_WEST, imageName: "zombie_0-0-0.png" },
		{ row: 0, direction: SPRITE_DIRECTION_NORTH_WEST, imageName: "zombie_0-0-1.png" },
		{ row: 0, direction: SPRITE_DIRECTION_NORTH, imageName: "zombie_0-0-2.png" },
		{ row: 0, direction: SPRITE_DIRECTION_NORTH_EAST, imageName: "zombie_0-0-3.png" },
		{ row: 0, direction: SPRITE_DIRECTION_EAST, imageName: "zombie_0-0-4.png" },
		{ row: 0, direction: SPRITE_DIRECTION_SOUTH_EAST, imageName: "zombie_0-0-5.png" },
		{ row: 0, direction: SPRITE_DIRECTION_SOUTH, imageName: "zombie_0-0-6.png" },
		{ row: 0, direction: SPRITE_DIRECTION_SOUTH_WEST, imageName: "zombie_0-0-7.png" }
	],

    tileHeight: 130,
    tileWidth: 128,
    drawWidth: 40,
    drawHeight: 60,
    initialDirection: SPRITE_DIRECTION_SOUTH_WEST
};
```
The idea behind this is that the json blobs would be stored on the server and requested as necessary via an ajax or jsonp loader, but that isn't quite ready yet and will be part of Traffic Cone server. 

### Inspecting and manipulating the isometric game world
Once you have sprites you'll obviously want to place them in the game world. You'll also want to know where they on screen and what cells are currently visible to the user's canvas (the view port). Ie, there's a whole big world that we have in memory, but we're only rendering what the user can see at any point in time. To handle this complexity for you, Traffic Cone provides an isometric game world model that can be fetched via:

```js
	var gameWorldModel = ga.getWorldModel();
```

The game world model then lets you do simple things like place sprites in world/screen space without having to think of all the funky angles that come with tilting a grid into fake 3d (2.5d?) space. For example, here is some code that places one of our zombies in the cell at the very center of our world.

```js
gameWorldModel.placeSpriteInCenterOfWorldCell((worldSize/2), worldSize/2, zom);
```
 
### Creating a simple composite sprite
The supporting code for this example can be seen in: assets/hero.js

What is a composite sprite? A composite sprite is a sprite that instead of being made of a single sprite sheet is instead made of multiple images. While I'm sure there are lots of good reasons for doing this the main reason is usually allowing some sort of customizable or permutable characters. For example changing equipment and weapons etc. While this is great you should be cognizant that internally Traffic Cone treats each successive layer of a sprite as it's own canvas draw, so having a composite sprite that consists of 3 sets of images == 3 draws and has near the performance impact of three sprites. So be war. 

Traffic Cone supports two different types of composite sprites Simple and Complex. Simple composites which we'll discuss here are called simple because they expect that the artist has been diligent and "lined up" all of the different composite frames so that when layered on top of one another they are in the correct place. As such, simple composites are easy to configure, but it also means that all of the images have to be the same size as the largest of them. This isn't the end of the world, but it means your artists have to be very cognizant about size and positioning and you can't tweak offsets in code. So it's a bit limiting. If you don't mind, read on. If not, skip to complex composites.

If you're still reading, then let's take a look at some simple composite sheets. Our "hero" sprite asset is a simple composite, so let's start there. In his sprite definition from SpriteInventory.js we most of "hero" is pretty much like our Zombie isometric sprite. Not a lot of surprises. 

```js
//hero
TCSpriteInventory["9e4a24cf-c760-43ec-abda-3ef2db22e2ab"] = {
    usage: USAGE_PC,
    prefix: "hero",
    speed: 2,
    imagePath: DOMAIN_PREFIX + "assets/hero/",
    behavior:
	{
	    foundState: "normal",
	    moveState: "walking",
	    moveInterval: 100,
	    definition: DOMAIN_PREFIX + "tccore/behaviors/OriginSeeker.js",
	    constructor: "OriginSeeker(behavior.moveState, behavior.foundState)",
	    eventType: "mousedown",
	    startState: "normal"
	},
    states:
	{
	    normal:
		{
		    name: "normal",
		    columnRange: { start: 0, end: 0 },
		    imageName: "male_base.png",
		    playState: -1,
		    speed: 200
		},
	    walking:
		{
		    name: "walking",
		    columnRange: { start: 0, end: 4 },
		    imageName: "male_base.png",
		    playState: 1,
		    speed: 200
		},
	    attacking:
		{
		    name: "attacking",
		    columnRange: { start: 4, end: 6 },
		    imageName: "male_base.png",
		    playState: 1,
		    speed: 100
		},
	    bow:
		{
		    name: "bow",
		    columnRange: { start: 5, end: 5 },
		    imageName: "male_base.png",
		    playState: 1,
		    speed: 100
		},
	    hit:
		{
		    name: "hit",
		    columnRange: { start: 6, end: 6 },
		    imageName: "male_base.png",
		    playState: 1,
		    speed: 100
		},
	    dead:
		{
		    name: "dead",
		    columnRange: { start: 7, end: 7 },
		    imageName: "male_base.png",
		    playState: -1,
		    speed: 300
		}
	},
    neutralState: "normal",
    rowDirectionMap:
	[
		{ row: 0, direction: SPRITE_DIRECTION_WEST },
		{ row: 1, direction: SPRITE_DIRECTION_NORTH_WEST },
		{ row: 2, direction: SPRITE_DIRECTION_NORTH },
		{ row: 3, direction: SPRITE_DIRECTION_NORTH_EAST },
		{ row: 4, direction: SPRITE_DIRECTION_EAST },
		{ row: 5, direction: SPRITE_DIRECTION_SOUTH_EAST },
		{ row: 6, direction: SPRITE_DIRECTION_SOUTH },
		{ row: 7, direction: SPRITE_DIRECTION_SOUTH_WEST }
	],
    tileHeight: 260,
    tileWidth: 267,
    drawWidth: 125,
    drawHeight: 145,
    initialDirection: SPRITE_DIRECTION_SOUTH_WEST,
    chain: ["male_heavy.png", "male_longsword.png", "male_shield.png"]
};
```

However, there are some differences here. First off, let's look take a look at https://github.com/j03m/trafficcone/blob/master/public/assets/hero/male_base.png. For this image, he is pretty much just a nude base model. So obviously we need to layer some stuff on him to flesh him out. Note that last portion of the definition: 

```js
chain: ["male_heavy.png", "male_longsword.png", "male_shield.png"]
```
Here we added a series of images that will be "chained" on top of the sprite. If you look at https://github.com/j03m/trafficcone/blob/master/public/assets/hero/male_heavy.png you can see it animates his armor allowing us to change out his clothes. In addition, male_shield + male_longsword animate his weapons allowing those to be swapped in and out as well. However, look at how much negative space is in use in male_longswrod. 

Under the hood, we're calling the sprite.chain method and sprite.unchain methods to add and remove images. Let's look at the implementation of TCSpriteFactory. On line 224 we can see that if the "chain" element of a sprite we end calling chain:

```js
//handle image chains if the sprite has them defined.    
 var chain = spriteDef.chain;
 if (chain != undefined) //chaining is optional
 {
     for (var i = 0; i < chain.length; i++) {
         sprite.chain(imagePath + chain[i]);
     }
 }
```

In our latest sample - examples/simpleCompositeEqSwap.html we can see here that chain and unchain can be called by you on the fly to modify a sprites equipment on the fly. You'll see another weakness of simple composites here as well, there is no concept of order. So you'll see in order to layer the sprite correctly we have re-chain the shield whenever we change the base armor. Complex composites handle this for you with the trade off of having to write more code and be more diligent about our definitions. 



### More to come - Docs Joe still needs to write: 
### Creating a complex composite sprite
Coming soon see: examples/CompositeSprite.html
### NPC Events and Controlling Input
Coming soon see: examples/2D or examples/Isometric.html
### Custom draw routines
Coming soon see: examples/CompositeSprite.html
### Frame level alerting
Coming soon see: examples/TrafficConeTests.html
### Collision detection
Coming soon see: examples/2D or examples/Isometric.html
### AI, Path Finding and Behaviors
Coming soon see: examples/Isometric.html
### Special sprite generators
Coming soon see: examples/CompositeSprite.html
### Slimming things down for mobile
Coming soon see: examples/IOSTextureExperiments.html - swaps out bg tiles for one textured background. Speeds up animations when moving the entire background is an issue.

===

### License:

Traffic Cone is licensed under the MIT license. For details please see:
http://en.wikipedia.org/wiki/MIT_License

Copyright (c) 2012 Joseph Mordetsky http://github.com/j03m

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
   