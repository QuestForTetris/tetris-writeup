## OTCA Metapixel

<img src="http://www.conwaylife.com/w/images/3/3a/Otcametapixel.png"/>
([Source](http://www.conwaylife.com/w/images/3/3a/Otcametapixel.png))

The [OTCA Metapixel](http://www.conwaylife.com/w/index.php?title=OTCA_metapixel) is a construct in Conway's Game of Life that can be used to simulate any Life-like cellular automata. As the LifeWiki (linked above) says,

> The OTCA metapixel is a 2048 × 2048 period 35328 unit cell that was constructed by Brice Due... It has many advantages...including the ability to emulate any Life-like cellular automaton and the fact that, when zoomed out, the ON and OFF cells are easy to distinguish...

What [Life-like cellular automata](http://www.conwaylife.com/wiki/Life-like_cellular_automaton#Life-like_cellular_automata) means here is essentially that cells are born and cells survive according to how many of their eight neighbor cells are alive. The syntax for these rules is as follows: a B followed by the numbers of live neighbors that will cause a birth, then a slash, then an S followed by the numbers of live neighbors that will keep the cell alive. A bit wordy, so I think an example will help. The canonical Game of Life can be represented by the rule B23/S3, which says that any dead cell with two or three live neighbors will become alive and any live cell with three neighbors will remain alive. Otherwise, the cell dies.

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

### Delay Components
