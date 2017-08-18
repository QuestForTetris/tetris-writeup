# Part 3: Hardware

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
We do this by using AND and ANT gates on the clock and parallel data.
However, we need to make sure that the parallel data is also outputted so that it can be compared again.
These are the gates that I came up with:

[![Signal Checking Gates](http://imgur.com/KAtnrKI.png)](http://play.starmaninnovations.com/varlife/nFweIfflLl)

Finally, we just split the clock signal, and stack a bunch of signal checkers (each one for a different address/output), and we have a multiplexer!

[![Multiplexer](http://imgur.com/hpgUufI.png)](http://play.starmaninnovations.com/varlife/JdsIIWIcPa)

## ROM
The ROM is supposed to take an address as an input, and send out the instruction at that address as output.
We start by using a multiplexer to direct the clock signal to one of the instructions.
Next, we need to generate a signal using some wire crossings and OR gates.
The wire crossings enable the clock signal to travel down all 58 bits of the instruction, and also allow for a generated signal (currently in parallel) to move down through the ROM to be outputted.

[![ROM bits](http://imgur.com/Nlj8B2F.png)](http://play.starmaninnovations.com/varlife/ZExrQPCgwf)

Next we just need to convert the parallel signal to serial data, and the ROM is complete.

[![Parallel to serial converter](http://imgur.com/lQCOFTm.png)](http://play.starmaninnovations.com/varlife/cPnZMldukU)

[![ROM](http://imgur.com/rwF6CL9.png)](http://play.starmaninnovations.com/varlife/gowVDURoIc)

## SRL, SL, SRA
These two logic gates are more complicated than your typical AND, OR, XOR, etc.
To make these gates work, we will just delay the clock signal an appropriate amount of time to cause a "shift" in the data.
The second argument given to these gates dictate how many bits to shift.
We just need to 1: make sure that the 12 most significant bits are not on (otherwise the output is simply 0), and 2: delay the data the right amount based on the 4 least significant bits.
This is easy with a bunch of AND/ANT gates, and a multiplexer.

![SRL](http://imgur.com/wtAkNw1.png)

The SRA is slightle different, because we need to copy the sign bit during the shift.
The way we do this is by ANDing the clock signal with the sign bit, and then copy that output a bunch of times with wire splitters and OR gates.

[![SRA](http://imgur.com/GwH8oTJ.png)](http://play.starmaninnovations.com/varlife/DdNReVfSua)

## Set-Reset (SR) latch
Many portions of the processor's functionality rely on the ability to store data.
Using 2 red B12/S1 cells, we can do just that.
The two cells can keep each other on, and can also stay off together.
Using some extra set, reset, and read circuitry, we can make a simple SR latch.

[![SR latch](http://imgur.com/W7eNmfr.png)](http://play.starmaninnovations.com/varlife/qiFFgGEvRd)

## Synchronizer
By converting serial data to parallel data, then setting a bunch of SR latches, we can store a whole word of data.
Then, to get the data out again, we can just read and reset all of the latches, and delay the data accordingly.

[![Synchronizer](http://imgur.com/fRgFuAR.png)](http://play.starmaninnovations.com/varlife/hYdaWoyjwz)

## Read Counter
This device keeps track of how many more times it needs to address from RAM.
It does this using a device similar to the SR latch: a T flip flop.
Every time the T flip flop recieves an input, it changes state: if it was on, it turns off, and vice versa.
When the T flip flop is turned from on to off, it gives an output pulse, which can be fed into another T flip flop to form a 2 bit counter.

[![Two bit counter](http://imgur.com/ayN556Y.png)](http://play.starmaninnovations.com/varlife/sqMCPQpoLS)

In order to make the Read Counter, we need to set the counter to the addressing mode with two ANT gates, and use the counter's output signal to decide where to direct the clock signal: to the ALU or to the RAM.

![Read Counter](http://imgur.com/Zf8t5PH.png)

## Read Queue
The read queue needs to keep track of which read counter sent an input to RAM, so that it can send the RAM's output to the correct location.
To do that, we use some SR latches.
When a signal is sent to RAM from a read counter, the clock signal is split and sets the counter's SR latch.
The RAM's output is then ANDed with the SR latch, and the clock signal from the RAM resets the SR latch.

![Read Queue](http://imgur.com/EkqUHae.png)

## ALU
The ALU functions similarly to the read queue, in that it uses an SR latch to keep track of where to send a signal.
First, the SR latch of the logic circuit corresponding to the opcode of the instruction is set using a multiplexer.
Next, the first and second argument's values are ANDed with the SR latch, and then are passed to the logic circuits.
The clock signal resets the latch as it's passing so that the ALU can be used again.
(Most of the circuitry is golfed down, and a ton of delay management is shoved in, so it looks like a bit of a mess)

![ALU](http://imgur.com/mC6tMoL.png)

## RAM
The RAM was one of the most complicated parts of this project.
It required for a very specific control over each SR latch that stored data.
For reading, the address is sent into a multiplexer and sent to the RAM units.
The RAM units output the data they store in parallel, which is converted to serial and outputted.
For writing, the address is sent into a different multiplexer, the data to be written is converted from serial to parallel, and the RAM units would propagte the signal throughout the RAM.
Each 22x22 metapixel RAM unit has this basic structure:

![RAM unit](http://imgur.com/zmjUg6p.png)

Putting the whole RAM together, we get something that looks like this:

![RAM](http://imgur.com/ytVtD1k.png)
