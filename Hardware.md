# Hardware

With our knowledge of logic gates and the general structure of the processor, we can start designing all the components of the computer.

## Demultiplexer
A demultiplexer, or demux, is a crucial component to the ROM, RAM, and ALU.
It routes a input signal to one of the many output signals based on what the selector data.
It is composed of 3 main parts: a serial to parallel converter, a signal checker, and a clock signal splitter.

We start with converting the serial selector data to "parallel."
This is done by strategically splitting and delaying the data so that the leftmost bit of data intersects the clock signal at the leftmost 11x11 square, the next bit of data intersects the clock signal the the next 11x11 square, and so on.
Although every bit of data will be outputted in every 11x11 square, every bit of data will intersect with the clock signal only once.

[![Serial to parallel converter](http://imgur.com/v6iX5d9.png)](http://play.starmaninnovations.com/varlife/YnGJhtTyGt)

Next, we will check to see if the parallel data matches a preset address.
We do this by using AND and ANT gates on the clock and parallel data
However, we need to make sure that the parallel data is also outputted so that it can be compared again.
These are the gates that I came up with:

[![Signal Checking Gates](http://imgur.com/KAtnrKI.png)](http://play.starmaninnovations.com/varlife/nFweIfflLl)

Finally, we just split the clock signal, and stack a bunch of signal checkers (each one for a different address/output), and we have a multiplexer!

[![Multiplexer](http://imgur.com/hpgUufI.png)](http://play.starmaninnovations.com/varlife/JdsIIWIcPa)

## ROM
The ROM is supposed to take an address as an input, and send out the instruction at that address as output.
We start by using a multiplexer to direct the clock signal to one of the instructions.
Next, we need to generate a signal using some wire crossings and OR gates.
The wire crossings enable the clock signal to travel down all 58 bits of the instruction, and also allow for a generated signal (currently in parallel) to move down through the ROM to be outputted

[![ROM bits](http://imgur.com/Nlj8B2F.png)](http://play.starmaninnovations.com/varlife/ZExrQPCgwf)

Next we just need to convert the parallel signal to serial data, and the ROM is complete.

[![Parallel to serial converter](http://imgur.com/lQCOFTm.png)](http://play.starmaninnovations.com/varlife/cPnZMldukU)

[![ROM](http://imgur.com/rwF6CL9.png)](http://play.starmaninnovations.com/varlife/gowVDURoIc)

## SRL, SL
These two logic gates are more complicated than your typical AND, OR, XOR, etc.
To make these gates work, we will just delay the clock signal an appropriate amount of time to cause a "shift" in the data
The second argument given to these gates dictate how many bits to shift.
We just need to 1: make sure that the 12 most significant bits are not on (otherwise the output is simply 0), and 2: delay the data the right amount based on the 4 least significant bits.
This is easy with a bunch of AND/ANT gates, and a multiplexer

![SRL](http://imgur.com/wtAkNw1.png)

## SR latch
There are many times during the designing of this computer where we need to store data.
Using 2 red B12/S1 cells, we can do just that.
The two cells can keep each other on, and can also stay off together.
Using some extra set, reset, and read circuitry, we can make a simple SR latch.

[![SR latch](http://imgur.com/W7eNmfr.png)](http://play.starmaninnovations.com/varlife/qiFFgGEvRd)

## Synchronizer
## Read Queue
## ALU
## RAM
