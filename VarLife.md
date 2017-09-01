## OTCA Metapixel

<img src="http://www.conwaylife.com/w/images/3/3a/Otcametapixel.png"/>
([Source](http://www.conwaylife.com/w/images/3/3a/Otcametapixel.png))

The [OTCA Metapixel](http://www.conwaylife.com/w/index.php?title=OTCA_metapixel) is a construct in Conway's Game of Life that can be used to simulate any Life-like cellular automata. As the LifeWiki (linked above) says,

> The OTCA metapixel is a 2048 × 2048 period 35328 unit cell that was constructed by Brice Due... It has many advantages...including the ability to emulate any Life-like cellular automaton and the fact that, when zoomed out, the ON and OFF cells are easy to distinguish...

What [Life-like cellular automata](http://www.conwaylife.com/wiki/Life-like_cellular_automaton#Life-like_cellular_automata) means here is essentially that cells are born and cells survive according to how many of their eight neighbor cells are alive. The syntax for these rules is as follows: a B followed by the numbers of live neighbors that will cause a birth, then a slash, then an S followed by the numbers of live neighbors that will keep the cell alive. A bit wordy, so I think an example will help. The canonical Game of Life can be represented by the rule B23/S3, which says that any dead cell with two or three live neighbors will become alive and any live cell with three neighbors will remain alive. Otherwise, the cell dies.

Despite being a 2048 x 2048 cell, the OTCA metapixel actually has a bounding box of 2058 x 2058 cells, the reason being that it overlaps by five cells in every direction with its neighbors. [[Why? What does this do? I have no idea.]] The birth and survival rules are encoded in a special section of cells at the left side of the metapixel, by the presence or absence of eaters in specific positions along two columns (one for birth, the other for survival). As for detecting the state of neighboring cells, here's how that happens:

> A 9-LWSS stream then goes clockwise around the cell, losing a LWSS for each adjacent ‘on’ cell that triggered a honeybit reaction. The number of missing LWSSes is counted by detecting the position of the front LWSS by crashing another LWSS into it from the opposite direction. This collision releases gliders, which triggers another one or two honeybit reactions if the eaters that indicate that birth/survival condition are absent.

A more detailed diagram of each aspect of the OTCA metapixel can be found at its original website: [How Does It Work?](http://otcametapixel.blogspot.com/2006/05/how-does-it-work.html).

## VarLife

I built an online simulator of Life-like rules where you could make any cell behave according to any life-like rule and called it "Variations of Life". This name has been shortened to "VarLife" to be more concise. Here's a screenshot of it (link to it here: http://play.starmaninnovations.com/varlife/BeeHkfCpNR):

[insert VarLife screenshot here]

Notable features:
 - Toggle cells between live/dead and paint the board with different rules.
 - The ability to start and stop the simulation, and to do one step at a time. It's also possible to do a given number of steps as fast as possible or more slowly, at the rate set in the ticks-per-second and milliseconds-per-tick boxes.
 - Clear all live cells or to entirely reset the board to a blank state.
 - Can change the cell and board sizes, and also to enable toroidal wrapping horizontally and/or vertically.
 - Permalinks (which encode all information in the url) and short urls (because sometimes there's just too much info, but they're nice anyway).
 - Rule sets, with B/S specification, colors, and optional randomness.
 - And last but definitely not least, rendering gifs!

The render-to-gif feature is my favorite both because it took a ton of work to implement, so it was really satisfying when I finally cracked it at 7 in the morning, and it makes it very easy to share VarLife constructs with others.

## Basic VarLife Circuitry

All in all, the VarLife computer only needs four cell types! Eight states in all counting the dead/alive states. They are:
 - B/S (black/white), which serves as a buffer between all components since B/S cells can never be alive.
 - B1/S (blue/cyan), which is the main cell type used to propagate signals.
 - B2/S (green/yellow), which is mainly used for signal control, ensuring it doesn't backpropagate.
 - B12/S1 (red/orange), which is used in a few specialized situations, such as crossing signals and storing a bit of data.
 
Use this short url to open up VarLife with these rules already encoded: [http://play.starmaninnovations.com/varlife/BeeHkfCpNR](http://play.starmaninnovations.com/varlife/BeeHkfCpNR).

### Wires

There are a few different wire designs with varying characteristics.

This is the easiest and most basic wire in VarLife. Simply a strip of blue bordered by strips of green.

[Remember to upload to SE Imgur]  
<img src="http://play.starmaninnovations.com/static/d3applets/renders/EMOLGwLAxh.gif"/>  
Short url: http://play.starmaninnovations.com/varlife/WcsGmjLiBF

This wire is unidirectional. That is, it will kill any signal attempting to travel in the opposite direction. It's also one cell narrower than the basic wire.

[Remember to upload to SE Imgur]  
<img src="http://play.starmaninnovations.com/static/d3applets/renders/AmoesLFnhV.gif"/>  
Short url: http://play.starmaninnovations.com/varlife/ARWgUgPTEJ

Diagonal wires also exist but they are not used much at all.

[Remember to upload to SE Imgur]  
<img src="http://play.starmaninnovations.com/static/d3applets/renders/yCJvsIHuMQ.gif"/>  
Short url: http://play.starmaninnovations.com/varlife/kJotsdSXIj

### Gates

There are actually a lot of ways to construct each individual gate, so I will only be showing one example of each kind. This first gif demonstrates AND, XOR, and OR gates, respectively. The basic idea here is that a green cell acts like an AND, a blue cell acts like an XOR, and a red cell acts like an OR, and all the other cells around them are just there to control the flow properly.

[Remember to upload to SE Imgur]  
<img src="http://play.starmaninnovations.com/static/d3applets/renders/ixoJIHLDPe.gif"/>  
Short url: http://play.starmaninnovations.com/varlife/EGTlKktmeI

The AND-NOT gate, abbreviated to "ANT gate", turned out to be a vital component. It is a gate that passes a signal from A if and only if there is a signal from B too.

[Remember to upload to SE Imgur]
<img src="http://play.starmaninnovations.com/static/d3applets/renders/RCPmrbcQIQ.gif"/>
Short url: http://play.starmaninnovations.com/varlife/RsZBiNqIUy

While not exactly a *gate*, a wire crossing tile is still very important and useful to have.

[Remember to upload to SE Imgur]  
<img src="http://play.starmaninnovations.com/static/d3applets/renders/dpvIZIdxMg.gif"/>  
Short url: http://play.starmaninnovations.com/varlife/OXMsPyaNTC

Incidentally, there is no NOT gate here. That's because without an incoming signal, a constant output must be produced, which does not work well with the variety in timings that the current computer hardware requires. We got along just fine without it anyway.

Also, many components were intentionally designed to fit within an 11 by 11 bounding box (a *tile*) where it takes signals 11 ticks from entering the tile to leave the tile. This makes components more modular and easier to slap together as needed without having to worry about adjusting wires for either spacing or timing.

To see more gates that were discovered/constructed in the process of exploring circuitry components, check out this blog post by PhiNotPi: [Building Blocks: Logic Gates](http://blog.phinotpi.com/2016/05/31/building-blocks-logic-gates/).

### Delay Components
