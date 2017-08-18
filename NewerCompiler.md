# The Newer Compiler to QFTASM
Although Cogol is sufficient for a rudimentary Tetris implementation, it is too simple and too low-level for general-purpose programming at an easily readable level.
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


Both the `while` and `if` low level interpreters are pretty simple: they get pointers to their conditions and jump depending on them. `while` loops are slightly different in that they're compiled as 


    ...
    condition
    jump to check
    code
    condition
    if condtion: jump to code
    ...

#### `call_sub` and `return`


Unlike most architectures, the computer we're compiling for doesn't have hardware support for a stack in the way of instructions for pushing and popping from a stack. This means that both pushing and popping from the stack take two instructions: Decrementing the stack pointer and copying the value to an address in the case of popping or copying a value from an address to the address at the current stack pointer and then incrementing it in the case of pushing.

All the locals for a subroutine are stored at a fixed location in RAM determined at compile time. In order to make recursion work, all the locals for a function are placed onto the stack at the start of a call. Then the arguments to the subroutine are copied to their position in the local store. The value of the return address is put onto the stack and the subroutine executes. When a `return` statement is encountered, the top of the stack is popped off and the program counter is set to that value. The values for the locals of the calling subroutine are the popped off the stack and into their previous position.

#### `assign`


Variable assignments are the easiest things to compile: they take a variable and a value and compile into the single line: `MLZ -1 VALUE VARIABLE`

### Assigning jump targets


Finally the compiler works out the jump targets for labels attached to instructions. The absolute position of labels is determined and then references to those labels are replaced with those values. Lables themselves are removed from the code and finally instuction numbers are added to the compiled code.

## Example step by step compilation


Now we've gone through all the stages, let's go through an actual compilation process for an actual program, step by step.
    
    #include stdint

    sub main
        a = 8
        b = 12
        c = a * b


Ok, simple enough. It should be obvious that at the end of the program, `a = 8`, `b = 12`, `c = 96`. Firstly, lets include the relevent parts of `stdint.txt`:

    operator (int a + int b) -> int
        return __ADD__(a, b)

    operator (int a - int b) -> int
        return __SUB__(a, b)

    operator (int a < int b) -> bool
        bool rtn = 0
        rtn = __MLZ__(a-b, 1)
        return rtn

    unsafe operator (int a * int b) -> int
        int rtn = 0
        for (int i = 0; i < b; i+=1)
            rtn += a
        return rtn

    sub main
        int a = 8
        int b = 12
        int c = a * b

Ok, slightly more complicated. Let's move onto the tokeniser and see what comes out. At this stage, we'll only have a linear flow of tokens without any form of structure

    NAME NAME operator
    LPAR OP (
    NAME NAME int
    NAME NAME a
    PLUS OP +
    NAME NAME int
    NAME NAME b
    RPAR OP )
    OP OP ->
    NAME NAME int
    NEWLINE NEWLINE
    INDENT INDENT     
    NAME NAME return
    NAME NAME __ADD__
    LPAR OP (
    NAME NAME a
    COMMA OP ,
    NAME NAME b
    RPAR OP )
    ...

Now all the tokens get put through the grammar parser and outputs a tree with the names of each of the sections. This shows the high level structure as read by the code.

    GrammarTree file
     'stmts': [GrammarTree stmts_0
      '_block_name': 'inline'
      'inline': GrammarTree inline
       '_block_name': 'two_op'
       'type_var': GrammarTree type_var
        '_block_name': 'type'
        'type': 'int'
        'name': 'a'
        '_global': False

       'operator': GrammarTree operator
        '_block_name': '+'

       'type_var_2': GrammarTree type_var
        '_block_name': 'type'
        'type': 'int'
        'name': 'b'
        '_global': False
       'rtn_type': 'int'
       'stmts': GrammarTree stmts
        ...
        
This grammar tree sets up information to be parsed by the high level compiler. It includes information such as structure types, attributes of a variable. It takes this information and assigns the variables that are needed, the subroutines. It also inserts all the inlines.

    ('sub', 'start', 'main')
    ('assign', int main_a, 8)
    ('assign', int main_b, 12)
    ('assign', int op(*:rtn), 0)
    ('assign', int op(*:i), 0)
    ('assign', global bool scratch_2, 0)
    ('call_sub', '__SUB__', [int op(*:i), int main_b], global int scratch_3)
    ('call_sub', '__MLZ__', [global int scratch_3, 1], global bool scratch_2)
    ('while', 'start', 1, 'for')
    ('call_sub', '__ADD__', [int op(*:rtn), int main_a], int op(*:rtn))
    ('call_sub', '__ADD__', [int op(*:i), 1], int op(*:i))
    ('assign', global bool scratch_2, 0)
    ('call_sub', '__SUB__', [int op(*:i), int main_b], global int scratch_3)
    ('call_sub', '__MLZ__', [global int scratch_3, 1], global bool scratch_2)
    ('while', 'end', 1, global bool scratch_2)
    ('assign', int main_c, int op(*:rtn))
    ('sub', 'end', 'main')

Next the low level compiler has to convert this high level representation into QFTASM code. Variables are assigned locations in RAM like so:

    int program_counter
    int op(*:i)
    int main_a
    int op(*:rtn)
    int main_c
    int main_b
    global int scratch_1
    global bool scratch_2
    global int scratch_3
    global int scratch_4
    global int <result>
    global int <stack>

The simple instructions are then compiled. Finally instruction numbers are added, resulting in executable QFTASM code.

    0. MLZ 0 0 0;
    1. MLZ -1 12 11;
    2. MLZ -1 8 2;
    3. MLZ -1 12 5;
    4. MLZ -1 0 3;
    5. MLZ -1 0 1;
    6. MLZ -1 0 7;
    7. SUB A1 A5 8;
    8. MLZ A8 1 7;
    9. MLZ -1 15 0;
    10. MLZ 0 0 0;
    11. ADD A3 A2 3;
    12. ADD A1 1 1;
    13. MLZ -1 0 7;
    14. SUB A1 A5 8;
    15. MLZ A8 1 7;
    16. MNZ A7 10 0;
    17. MLZ 0 0 0;
    18. MLZ -1 A3 4;
    19. MLZ -1 -2 0;
    20. MLZ 0 0 0;
