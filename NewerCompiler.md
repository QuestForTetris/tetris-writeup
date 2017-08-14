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
The grammar needed to support repeated nodes as well as optional ones. This was achieved by introducing meta tags to describe how tokens were to be read.
The tokens that are generated then get parsed through the rules of the grammar such that the output forms a tree of grammar elements such as `sub`s and `generic_variables`, which in turn contain other grammar elements and tokens.


## Compilation into high level code
Each feature of the language needs to be able to be compiled into high level constructs. These include `assign(a, 12)`
 and `call_subroutine(is_prime, call_variable=12, return_variable=temp_var)`. Features such as the inlining of elements are executed in this segmant. These are definined as `operator`s and are special in that they're inlined every time an operator such as `+` or `%` are used. Because of this, they're more restricted than regular code - they can't use their own operator nor any operator that relies on the one being defined. During the inlining process, the internal variables are replaced with the ones being called. This in effect turns
 
 
    operator(int a + int b) -> int c
        return __ADD__(a, b)
    int i = 3+3
  
into


    int i = __ADD__(3, 3)
  
This behaviour however can be detrimental and bug prone if the input variable and output variables point to the same location in memory. To use 'safer' behaviour, the `unsafe` keyword adjusts the compilation process such that additional variables are created and copied to and from the inline as needed.

### Scratch variables and complex operations


Mathematical operations such as `a += (b + c) * 4` cannot be calculated without using extra memory cells. The high level compiler deals with this by seperating the operations into different sections:


    scratch_1 = b + c
    scratch_1 = scratch_1 * 4
    a = a + scratch_1
    
This introduces the concept of scratch variables which are used for storing intermediate information of calculations. They're allocated as required and deallocated into the general pool once finished with. This decreases the number of scratch memory locations required for use. Scratch variables are considered globals.

Each subroutine has it's own VariableStore to keep a reference to all of the variables the subroutine uses as well as their type. At the end of the compilation, they're translated into relative offsets from the start of the store and then given actual addresses in RAM.

### RAM Structure


    Program counter
    Subroutine locals
    Operator locals (reused throughout)
    Scratch variables
    Result variable
    Stack pointer
    Stack
    ...

## Low level compilation


The only things the low level compiler has to deal with are `sub`, `call_sub`, `return`, `assign`, `if` and `while`. This is a much reduced list of tasks that can be translated into QFTASM instructions more easily.

#### `sub`


This locates the start and end of a named subroutine. The low level compiler adds labels and in the case of the `main` subroutine, adds an exit instruction (jump to end of ROM).

#### `if` and `while`


...

#### `call_sub` and `return`


...

#### `assign`


...

### Assigning jump targets


...
