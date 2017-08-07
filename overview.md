1. Use GoL to create [metapixels](http://www.conwaylife.com/wiki/OTCA_metapixel) which are capable of emulating any "life-like" rule.  This means that any particular metapixel is in one of two possible states, and its rule is defined by birth and survival rules.

2. Use these metapixels to create something akin to [Wireworld](https://en.wikipedia.org/wiki/Wireworld).  Wireworld is notable due to the existence of the [Wireworld computer](http://www.quinapalus.com/wi-index.html), another example of a processor built in a cellular automatan.  Although metapixels cannot emulate Wireworld directly, a key insight is that different metapixels can be assigned different rules.  For our computer we used B1/S, B2/S, and B12/S1 cells on an empty background.

3. Create a set of useful logical components, each of which is the same size (the size can be arbitrary, as smaller components can have their I/O wires extended to make them "larger"), and each of which have the same signal delay.  This means that, if the component fits in a 20x20 square, there must be exactly 20 generations between when the signals enter and leave.  This way, the delay of each component (including wire) is uniform, and then we can treat them as "tileable".

4. Design a processor architecture suitable for programming Tetris.  Here, we spent significant effort on designing an architecture that was both as non-esoteric and as easily-implementable as possible.  Whereas the Wireworld computer used a rudimentary transport-triggered architecture, this project uses a much more flexible RISC architecture complete with multiple opcodes and addressing modes.

5. Use the logic components to create circuits and procesor components, including ROM, RAM, and the ALU.  All of the effort we spent on small-scale timing should mean that timing is a lot easier to calculate at this stage.  

6. Create a compiler or other method to convert from a higher-level language to QFTASM.

7. Program Tetris.

8. Assemble the finished processor with the Tetris ROM and generate the final GoL state.
