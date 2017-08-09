*This began as a quest but ended as an odyssey.*

## Quest for Tetris Processor, (insert bounding box size here)

This project is the culmination of the efforts of many users over the course of the past 1 & 1/2 years.  Although the composition of the team has varied over time, the participants as of writing are the following:

- PhiNotPi
- El'endia Starman
- K Zhang
- Muddyfish
- Kritixi Lithos
- Mego
- Quartata

We would also like to extend our thanks to 7H3_H4CK3R, Conor O'Brien, and the many other users who have put effort into solving this challenge.

Due to the unprecedented scope of this collaboration, this answer is split in parts across multiple answers written by the members of this team.  Each member will write about specific sub-topics, roughly corresponding to the areas of the project in which they were most involved.

***Please distribute any upvotes or bounties across all members of the team.***

## Part 1: Overview

The underlying idea of this project is *abstraction*.  Rather than develop a Tetris game in Life directly, we slowly ratcheted up the abstraction in a series of steps.  At each layer, we get further away from the difficulties of Life and closer to the construction of a computer that is as easy to program as any other.

First, we used [OTCA metapixels](http://www.conwaylife.com/wiki/OTCA_metapixel) as the foundation of our computer. These metapixels are capable of emulating any "life-like" rule.  [Wireworld](https://en.wikipedia.org/wiki/Wireworld) and the [Wireworld computer](http://www.quinapalus.com/wi-index.html) served as important sources of inspiration for this project, so we sought to create a similar constuction with metapixels.  Although it is not possible to emulate Wireworld with OTCA metapixels, it is possible to assign different metapixels different rules and to build metapixel arrangements that function similarly to wires.

The next step was to construct a variety of fundamental logic gates to serve as the basis for the computer.  Already at this stage we are dealing with concepts similar to real-world processor design.  Here is an example of an OR gate, each cell in this image is actually an entire OTCA metapixel.  You can see "electrons" (each representing a single bit of data) enter and leave the gate.  You can also see all of the different metapixel types that we used in our computer: B/S as the black background, B1/S in blue, B2/S in green, and B21/S1 in red.

![image][1]

[1]: http://play.starmaninnovations.com/static/d3applets/renders/cieVczzgcc.gif

From here we developed an architecture for our processor.  We spent significant effort on designing an architecture that was both as non-esoteric and as easily-implementable as possible.  Whereas the Wireworld computer used a rudimentary transport-triggered architecture, this project uses a much more flexible RISC architecture complete with multiple opcodes and addressing modes.  We created an assembly language, known as QFTASM (Quest for Tetris Assembly), which guided the construction of our processor.

Our computer is also asynchronous, meaning that there is no global clock controlling the computer.  Rather, the data is accompanied by a clock signal as it flows around the computer, which means we only need to focus on local but not global timings of the computer.

Here is an illustration of our processor architecture:

![image][2]

[2]: https://i.stack.imgur.com/JXmnf.png

From here it is just a matter of implementing Tetris on the computer.  To help accomplish this, we have worked on multiple methods of compiling higher-level language to QFTASM.  We have a basic language called Cogol, a second, more advanced language under development, and finally we have an under-construction LLVM backend.  The current Tetris program was written in / compiled from Cogol.

Once the final Tetris QFTASM code was generated, the final steps were to assemble from this code to corresponding ROM, and then from metapixels to the underlying Game of Life, completing our construction.

### Running Tetris

For those who wish to play Tetris without messing around with the computer, you can run the [Tetris source code](https://github.com/QuestForTetris/Cogol/blob/master/tetris.qftasm) on the [QFTASM interpreter](http://play.starmaninnovations.com/qftasm/).  Set the RAM display addresses to 3-34 to view the entire game.

Game features:

- All 7 tetrominoes
- Movement, rotation, soft drops
- Line clears and scoring
- Preview piece
- Player inputs inject randomness

**Display**

Our computer represents the Tetris board as a grid within its memory.  Addresses 10-31 display the board, addresses 5-8 display the preview piece, and address 3 contains the score. 

**Input**

Input to the game is performed by manually editing the contents of RAM address 1.  Using the QFTASM interpreter, this means performing direct writes to address 1.  Each move only requires aditing a single bit of RAM, and this input register is automatically cleared after the input event has been read.

    value     motion
       1      counterclockwise rotation
       2      left
       4      down (soft drop)
       8      right
      16      clockwise rotation

**Scoring system**

You get a bonus for clearing multiple lines in a single turn.

    1 row    =  1 point
    2 rows   =  2 points
    3 rows   =  4 points
    4 rows   =  8 points
