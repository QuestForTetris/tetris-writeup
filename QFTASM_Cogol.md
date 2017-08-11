## Part 4: QFTASM and Cogol

### Architecture Overview

In short, our computer has a 16-bit asynchronous RISC Harvard architecture.  When building a processor by hand, a RISC ([reduced instruction set computer](https://en.wikipedia.org/wiki/Reduced_instruction_set_computer)) architecture is practically a requirement.  In our case, this means that the number of opcodes is small and, much more importantly, that all instructions are processed in a very similar manner.

For reference, the wireworld computer used a [transport-triggered architecture](https://en.wikipedia.org/wiki/Transport_triggered_architecture), in which the only instruction was `MOV` and computations were performed by writing/reading special registers.  Although this paradigm leads to a very easy-to-implement architecture, the result is also borderline unusable: all arithmetic/logic/conditional operations require *three* instructions.  It was clear to us that we wanted to create a much less esoteric architecture.

in order to keep our processor simple while increasing usability, we made several important design decisions:

- No registers.  Every address in RAM is treated equally and can be used as any argument for any operation.  In a sense, this means all of RAM could be treated like registers.  This means that there are no special load/store instructions.
- In a similar vein, memory-mapping.  Everything that could be written to or read from shares a unified addressing scheme.  This means that the program counter (PC) is address 0, and the only difference between regular instructions and control-flow instructions is that control-flow instructions use address 0.
- Data is serial in transmission, parallel in storage.  Due to the "electron"-based nature of our computer, addition and subtraction are significantly easier to implement when the data is transmitted in serial little-endian (least-significant bit first) form.  Furthermore, serial data removes the need for cumbersome data buses, which are both really wide and cumbersome to time properly (in order for the data to stay together, all "lanes" of the bus must experience the same travel delay).
- Harvard architecture, meaning a division between program memory (ROM) and data memory (RAM).  Although this does reduce the flexibility of the processor, this helps with size optimization: the length of the program is much larger than the amount of RAM we'll need, so we can split the program off into ROM and then focus on compressing the ROM, which is much easier when it is read-only.
- 16-bit data width.  This is the smallest power of two that is wider than a standard Tetris board (10 blocks).  This gives us a data range of -32768 to +32767 and a maximum program length of 65536 instructions. (2^8=256 instructions is enough for most simple things we might want a toy processor to do, but not Tetris.)
- Asynchronous design.  Rather than having a central clock (or, equivalently, several clocks) dictating the timing of the computer, all data is accompanied by a "clock signal" which travels in parallel with the data as it flows around the computer.  Certain paths may be shorter than others, and while this would pose difficulties for a centrally-clocked design, 
- All instructions are of equal size.  We felt that an architecture in which each instruction has 1 opcode with 3 operands (value value destination) was the most flexible option.  This encompasses binary data operations as well as conditional movies.
- Simple addressing modes.  Addressing modes can be a source of significant complexity for a processor like this, as many real-world addressing modes involve multi-step computations (like adding offsets).  We also sought to unify the concepts of using hard-coded numbers as operands and using data addresses as operands.  This led to the creation of counter-based addressing modes: the addressing mode of an operand is a number representing how many times the data should be sent around a RAM read loop.  This encompasses immediate, direct, indirect, and double-indirect addressing.

An illustration of our architecture is contained in the overview post.

### Functionality and ALU Operations

From here, it was a matter of determining what functionality our processor will have.  Special attention was paid to the ease of implementation as well as the versatility of each command.

**Conditional Moves**

Conditional moves are very important and serve as both small-scale and large-scale control flow.  "Small-scale" refers to its ability to control the execution of a particular data move, while "large-scale" refers to its use as a conditional jump operation to transfer control flow to any arbitrary piece of code.  There are no dedicated jump operations because, due to memory mapping, a conditional move can both copy data to regular RAM and copy a destination address to the PC.  We also chose to forgo both unconditional moves and unconditional jumps for a similar reason: both can be implemented as a conditional move with a condition that's hardcoded to TRUE.

We chose to have two different types of conditional moves: "move if not zero" (`MNZ`) and "move if less than zero" (`MLZ`).  Funtionally, `MNZ` amounts to checking whether any bit in the data is a 1, while `MLZ` amounts to checking if the sign bit is 1.  They are useful for equalities and comparisons, respectfully.  The reason we chose these two over others such as "move if zero" (`MEZ`) or "move if greater than zero" (`MGZ`) was that `MEZ` would require creating a TRUE signal from an empty signal, while `MGZ` is a more complex check, requiring the the sign bit be 0 while at least one other bit be 1.

**Arithmetic**

The next-most important instructions, in terms of guiding the processor design, are the basic arithmetic operations. As I mentioned earlier, we are using little-endian serial data, with the choice of endianness determined by the ease of addition/subtraction operations.  By having the least-significant bit arrive first, the arithmetic units can easily keep track of the carry bit.

We chose to use 2's complement representation for negative numbers, since this makes addition and subtraction more consistent.  It's worth noting that the wireworld computer used 1's complement.

Addition and subtraction are the extent of our computer's native arithmetic support (besides bitshifts which are discussed later).  Other operations, like multiplication, are far too complex to be handled by our architecture, and must be implemented in software.

**Bitwise Operations**

Our processor has `AND`, `OR`, and `XOR` instructions which do as you would expect.  Rather than have a `NOT` instruction, we chose to have an "and-not" (`ANT`) instruction.  The difficulty with the `NOT` instruction is again that it must create signal from a lack of signal, which is difficult with a cellular automata.  The `ANT` instruction returns 1 only if the first argument bit is 1 and the second argument bit is 0.  Thus, `NOT x` is equivalent to `ANT -1 x` (as well as `XOR -1 x`).  Furthermore, `ANT` is versatile and has its main advantage masking: in the case of the Tetris program we use it to erase tetronimoes.

**Bit Shifting**

The bit-shifting operations are the most complex operations handled by the ALU.  They take two datta inputs: a value to shift and an amount to shift it by.  Despite their complexity (due to the variable amount of shifting), these operations are crucial for many important tasks, including the many "graphical" operations involved in Tetris.  Bit shifts would also serve as the foundation for efficient multiplication/dividion algorithms.

Our processor has two bit shift operations, "shift left" (`SL`) and "shift right logical" (`SRL`).  These bit shifts fill in the new bits with all zeros (meaning that a negative number shifted right will no longer be negative).  So far we have found these shifts to be sufficient.

There were plans for a "shift right arithmetic" (`SRA`) operation which preserved the sign of the input, and therefore acted as a true division by two, but this plan was cancelled due to the difficulty of implementation.

### Pipelining

### Quest for Tetris Assembly

### Cogol
