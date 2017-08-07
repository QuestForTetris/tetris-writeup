# The Newer Compiler to QFTASM
It was decided that COGOL was a bit too low level to create Tetris in readably.
A new language started being created in September 2016.
Progress on the language was slow due to hard to understand bugs as well as real life.
We built a low level language with similar syntax to Python, including a simple type system, subroutines supporting recursion and inline operators.
The compiler from text to QFTASM was created with 4 steps: the tokeniser, the grammar tree, a high level compiler and a low level compiler.

## The tokeniser
Development was started using Python using the built in tokeniser library, meaning this step was pretty simple.
Only a few changes to the default output were required, including stripping comments (but not `#include`s).


## The grammar tree
The grammar tree was created to be easily extendible without having to modify any source code.
The tree structure is stored in an XML file which includes the structure of the nodes that can make up the tree and how they're made up with other nodes and tokens.
The grammar needed to support repeated nodes as well as optional ones.
