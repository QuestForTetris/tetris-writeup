# Assembly and Translation

With our assembly program from the compiler, it's time to assemble a ROM for the Varlife computer, and translate everything into a big GoL pattern!

## Assembly

Assembling the assembly program into a ROM is done in much the same way as in traditional programming: each instruction is translated into a binary equivalent, and those are then concatenated into a large binary blob that we call an executable. For us, the only difference is, that binary blob needs to be translated into Varlife circuits and attached to the computer.

K Zhang wrote [CreateROM.py](https://github.com/QuestForTetris/QFT/blob/master/CreateROM.py), a Python script for Golly that does the assembly and translation. It's fairly straightforward: it takes an assembly program from the clipboard, assembles it into a binary, and translates that binary into circuitry. Here's an example with a simple primality tester included with the script:

```
#0. MLZ -1 3 3;
#1. MLZ -1 7 6; preloadCallStack
#2. MLZ -1 2 1; beginDoWhile0_infinite_loop
#3. MLZ -1 1 4; beginDoWhile1_trials
#4. ADD A4 2 4;
#5. MLZ -1 A3 5; beginDoWhile2_repeated_subtraction
#6. SUB A5 A4 5;
#7. SUB 0 A5 2;
#8. MLZ A2 5 0;
#9. MLZ 0 0 0; endDoWhile2_repeated_subtraction
#10. MLZ A5 3 0;
#11. MNZ 0 0 0; endDoWhile1_trials
#12. SUB A4 A3 2;
#13. MNZ A2 15 0; beginIf3_prime_found
#14. MNZ 0 0 0;
#15. MLZ -1 A3 1; endIf3_prime_found
#16. ADD A3 2 3;
#17. MLZ -1 3 0;
#18. MLZ -1 1 4; endDoWhile0_infinite_loop
```

This produces the following binary:

```
0000000000000001000000000000000000010011111111111111110001
0000000000000000000000000000000000110011111111111111110001
0000000000000000110000000000000000100100000000000000110010
0000000000000000010100000000000000110011111111111111110001
0000000000000000000000000000000000000000000000000000000000
0000000000000000000000000000000011110100000000000000100000
0000000000000000100100000000000000110100000000000001000011
0000000000000000000000000000000000000000000000000000000000
0000000000000000000000000000000000110100000000000001010001
0000000000000000000000000000000000000000000000000000000001
0000000000000000000000000000000001010100000000000000100001
0000000000000000100100000000000001010000000000000000000011
0000000000000001010100000000000001000100000000000001010011
0000000000000001010100000000000000110011111111111111110001
0000000000000001000000000000000000100100000000000001000010
0000000000000001000000000000000000010011111111111111110001
0000000000000000010000000000000000100011111111111111110001
0000000000000001100000000000000001110011111111111111110001
0000000000000000110000000000000000110011111111111111110001
```

When translated to Varlife circuits, it looks like this:

![ROM](http://i.imgur.com/eoPdvOY.png) ![closeup ROM](http://i.imgur.com/Pee1yuE.png)

The ROM is then linked up with the computer, which forms a fully functioning program in Varlife. But we're not done yet...

## Translation to Game of Life

This whole time, we've been working in various layers of abstraction above the base of Game of Life. But now, it's time to pull back the curtain of abstraction and translate our work into a Game of Life pattern. As previously mentioned, we are using the [OTCA Metapixel](http://www.conwaylife.com/w/index.php?title=OTCA_metapixel) as the base for Varlife. So, the final step is to convert each cell in Varlife to a metapixel in Game of Life.

Thankfully, Golly comes with a script ([metafier.py](https://sourceforge.net/p/golly/code/ci/master/tree/Scripts/Python/metafier.py)) that can convert patterns in different rulesets to Game of Life patterns via the OTCA Metapixel. Unfortunately, it is only designed to convert patterns with a single global ruleset, so it doesn't work on Varlife. I wrote a [modified version](https://github.com/QuestForTetris/QFT/blob/master/metafier.py) that addresses that issue, so that the rule for each metapixel is generated on a cell-by-cell basis for Varlife.

So, our computer (with the Tetris ROM) has a bounding box of 1,436 x 5,082. Of the 7,297,752 cells in that box, 6,075,811 are empty space, leaving an actual population count of 1,221,941. Each of those cells needs to be translated into an OTCA metapixel, which has a bounding box of 2048x2048 and a population of either 64,691 (for an ON metapixel) or 23,920 (for an OFF metapixel). That means the final product will have a bounding box of 2,940,928 x 10,407,936 (plus a few thousand extra for the borders of the metapixels), with a population between 29,228,828,720 and 79,048,585,231. With 1 bit per live cell, that's between 27 and 74 GiB needed to represent the entire computer and ROM.

I included those calculations here because I neglected to run them before starting the script, and very quickly ran out of memory on my computer. After a panicked `kill` command, I made a modification to the metafier script. Every 10 lines of metapixels, the pattern is saved to disk (as a gzipped RLE file), and the grid is flushed. This adds extra runtime to the translation and uses more disk space, but keeps memory usage within acceptable limits. Because Golly uses an extended RLE format that includes the location of the pattern, this doesn't add any more complexity to the loading of the pattern - just open all of the pattern files on the same layer.

K Zhang built off of this work, and created a [more efficient metafier script](https://github.com/QuestForTetris/QFT/blob/master/MetafierV2.py) that utilizes the MacroCell file format, which is loads more efficient than RLE for large patterns. This script runs considerably faster (a few seconds, compared to multiple hours for the original metafier script), creates vastly smaller output (121 KB versus 1.7 GB), and can metafy the entire computer and ROM in one fell swoop without using a massive amount of memory. It takes advantage of the fact that MacroCell files encode trees that describe the patterns. By using a custom template file, the metapixels are pre-loaded into the tree, and after some computations and modifications for neighbor detection, the Varlife pattern can simply be appended.

The pattern file of the entire computer and ROM in Game of Life can be found [here](https://github.com/QuestForTetris/QFT/blob/master/TetrisOTCAMP.mc.gz).

---

# The Future of the Project

Now that we've made Tetris, we're done, right? Not even close. We have several more goals for this project that we are working towards:

- muddyfish and kritixi lithos (?) are continuing work on the higher-level language that compiles to QFTASM.
- El'endia Starman is working on upgrades to the online QFTASM interpreter, including implementing permalinks.
- quartata is working on a LLVM backend for the QFT computer, which will allow us to compile any program in any language that LLVM supports and output a ROM that the computer can use. Currently, he has hit a roadblock, and has switched over to working on a GCC backend (which is simpler, but supports fewer languages). Once he finishes one of those backends, I will be working on porting some more-complex programs.
- One of the biggest hurdles that we have to overcome before we can make more progress is the fact that our tools can't emit position-independent code (e.g. relative jumps). Without PIC, we can't do any linking, and so we miss out on the advantages that come from being able to link to existing libraries. We're working on trying to find a way to do PIC correctly.
- We are discussing the next program that we want to write for the QFT computer. Right now, Pong looks like a nice goal.
