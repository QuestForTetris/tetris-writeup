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
- All instructions are of equal size.  We felt that an architecture in which each instruction has 1 opcode with 3 operands (value value destination) was the most flexible option.  This encompasses binary data operations as well as conditional moves.
- Simple addressing mode system.  Having a variety of addressing modes is very useful for supporting things such as arrays or recursion.  We managed to implement several important addressing modes with a relatively simple system.

An illustration of our architecture is contained in the overview post.

### Functionality and ALU Operations

From here, it was a matter of determining what functionality our processor should have.  Special attention was paid to the ease of implementation as well as the versatility of each command.

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

### Instruction Pipelining

Now's the time to talk about some of the gritty details of the architecture. Each CPU cycle consists of the following five steps:

**1. Fetch the current instruction from ROM**

The current value of the PC is used to fetch the corresponding instruction from ROM.  Each instruction has one opcode and three operands.  Each operand consisted of one data word and one addressing mode.  These parts are split from each other as they are read from the ROM.

The opcode is 4 bits to support 16 unique opcodes, of which 10 are assigned:

    0000  MNZ    Move if Not Zero
    0001  MLZ    Move if Less than Zero
    0010  ADD    ADDition
    0011  SUB    SUBtraction
    0100  AND    bitwise AND
    0101  OR     bitwise OR
    0110  XOR    bitwise eXclusive OR
    0111  ANT    bitwise And-NoT
    1000  SL     Shift Left
    1001  SRL    Shift Right Logical
    1010  reserved for SRA (Shift Right Arithmetic)
    1011  unassigned
    1100  unassigned
    1101  unassigned
    1110  unassigned
    1111  unassigned

**2. Write the result (if necessary) of the *previous* instruction to RAM**

Depending on the condition of the previous instruction (such as the value of the first argument for a conditional move), a write is performed.  The address of the write is determined by the third operand of the previous instruction.

It is important to note that writing occurs after instruction fetching.  This leads to the creation of a [branch delay slot](https://en.wikipedia.org/wiki/Delay_slot#Branch_delay_slots) in which the instruction immediately after a branch instruction (any operation which writes to the PC) is executed in lieu of the first instruction at the branch target.

In certain instances (like unconditional jumps), the branch delay slot can be optimized away.  In other cases it cannot, and the instruction after a branch must be left empty.  Furthermore, this type of delay slot means that branches must use a branch target that is 1 address less than the actual target instruction, to account for the PC increment that occurs.

**3. Read the data for the current instruction's arguments from RAM**

As mentioned earlier, each of the three operands consists of both a data word and an addressing mode.  The data word is 16 bits, the same width as RAM.  The addressing mode is 2 bits.

Addressing modes can be a source of significant complexity for a processor like this, as many real-world addressing modes involve multi-step computations (like adding offsets).  At the same time, versatile addressing modes play an important role in the usability of the processor.

We sought to unify the concepts of using hard-coded numbers as operands and using data addresses as operands.  This led to the creation of counter-based addressing modes: the addressing mode of an operand is simply a number representing how many times the data should be sent around a RAM read loop.  This encompasses immediate, direct, indirect, and double-indirect addressing.

    00  Immediate:  A hard-coded value. (no RAM reads)
    01  Direct:  Read data from this RAM address. (one RAM read)
    10  Indirect:  Read data from the address given at this address. (two RAM reads)
    11  Double-indirect: Read data from the address given at the address given by this address. (three RAM reads)

After this dereferencing is performed, the three operands of the instruction have different roles.  The first operand is usually the first argument for a binary operator, but also serves as the condition when the current instruction is a conditional move.  The second operand serves as the second argument for a binary operator.  The third operand serves as the destination address for the result of the instruction.

Since the first two instructions serve as data while the third serves as an address, the addressing modes have slightly different interpretations depending on which position they are used in.  For example, the direct mode is used to read data from a fixed RAM address (since one RAM read is needed), but the immediate mode is used to write data to a fixed RAM address (since no RAM reads are necessary).
  
**4. Compute the result**

The opcode and the first two operands are sent to the ALU to perform a binary operation.  For the arithmetic, bitwise, and shift operations, this means performing the relevant operation.  For the conditional moves, this means simply returning the second operand.

The opcode and first operand are used to compute the condition, which determines whether or not to write the result to memory.  In the case of conditional moves, this means either determining whether any bit in the operand is 1 (for `MNZ`), or determining whether the sign bit is 1 (for `MLZ`).  If the opcode isn't a conditional move, then write is always performed (the condition is always true).

**5. Increment the program counter**

Finally, the program counter is read, incremented, and written.

Due to the position of the PC increment between the instruction read and the instruction write, this means that an instruction which increments the PC by 1 is a no-op.  An instruction that copies the PC to itself causes the next instruction to be executed twice in a row.  But, be warned, multiple PC instructions in a row can cause complex effects, including infinite looping, if you don't pay attention to the instruction pipeline.

### Quest for Tetris Assembly

We created a new assembly language named QFTASM for our processor.  This assembly language corresponds 1-to-1 with the machine code in the computer's ROM.

Any QFTASM program is written as a series of instructions, one per line.  Each line is formatted like this:

    [line numbering] [opcode] [arg1] [arg2] [arg3]; [optional comment]

**Opcode List**

As discussed earlier, there are ten opcodes supported by the computer, each of which have three operands:

    MNZ [test] [value] [dest]  – Move if Not Zero; sets [dest] to [value] if [test] is not zero.
    MLZ [test] [value] [dest]  – Move if Less than Zero; sets [dest] to [value] if [test] is less than zero.
    ADD [val1] [val2] [dest]   – ADDition; store [val1] + [val2] in [dest].
    SUB [val1] [val2] [dest]   – SUBtraction; store [val1] - [val2] in [dest].
    AND [val1] [val2] [dest]   – bitwise AND; store [val1] & [val2] in [dest].
    OR [val1] [val2] [dest]    – bitwise OR; store [val1] | [val2] in [dest].
    XOR [val1] [val2] [dest]   – bitwise XOR; store [val1] ^ [val2] in [dest].
    ANT [val1] [val2] [dest]   – bitwise And-NoT; store [val1] & (![val2]) in [dest].
    SL [val1] [val2] [dest]    – Shift Left; store [val1] << [val2] in [dest].
    SRL [val1] [val2] [dest]   – Shift Right Logical; store [val1] >>> [val2] in [dest]. Doesn't preserve sign.

There was also an 11th planned but unimplemented instruction.  This instruction, however, is still supported by interpreters.

    SRA [val1] [val2] [dest]   – Shift Right Arithmetic; store [val1] >> [val2] in [dest], while preserving sign.

**Addressing Modes**

Each of the operands contains both a data value and an addressing move.  The data value is described by a decimal number in the range -32768 to 65536.  The addressing mode is described by a one-letter prefix to the data value.

    mode    name               prefix
    0       immediate          (none)
    1       direct             A
    2       indirect           B
    3       double-indirect    C 

**Example Code**

Fibonacci sequence in five lines:

    0. MLZ -1 1 1;    initial value
    1. MLZ -1 A2 3;   start loop, shift data
    2. MLZ -1 A1 2;   shift data
    3. MLZ -1 0 0;    end loop
    4. ADD A2 A3 1;   branch delay slot, compute next term

This code computes the Fibonacci sequence, with RAM address 1 containing the current term.  It quickly overflows after 28657.

Gray code:

    0. MLZ -1 5 1;      initial value for RAM address to write to
    1. SUB A1 5 2;      start loop, determine what binary number to covert to Gray code
    2. SRL A2 1 3;      shift right by 1
    3. XOR A2 A3 A1;    XOR and store Gray code in destination address
    4. SUB B1 42 4;     take the Gray code and subtract 42 (101010)
    5. MNZ A4 0 0;      if the result is not zero (Gray code != 101010) repeat loop
    6. ADD A1 1 1;      branch delay slot, increment destination address

This code computes Gray code and stores the code in succesive addresses starting at address 5.  This program utilizes several important features such as indirect addressing and a conditional jump.

### Online Interpreter

El'endia Starman has created a very useful online interpreter [here](http://play.starmaninnovations.com/qftasm/).  You are able to step through the code, set breakpoints, perform manual writes to RAM, and visualize the RAM as a display.

### Cogol
