# The Newer Language

(todo: fluff/introduction, probably give a name to the language, also probably restructure everything)

As stated by muddyfish, a newer compiler was made to be more high-leveled than COGOL. So here I will show you some of the features of this language.

## Operators

(todo probably send this to the end, after subroutines)

First up, this language has custom operators. A file containing some of the standard operators has been created called `stdint.txt`. This can be included in a [language-name] program using `#include stdint`. Now this list of operators do not jsut contain direct translations to opcodes (`+` to `__ADD__`) but also more complex ones such as operators for multiplication and modulo (both of which are not QFTASM opcodes). Additionally, one can use the opcodes directly by using `__OPCODENAME__`.


Here is an example operator that performs the modulo operation. This uses a simple long division algorithm. Note that an operator would be `unsafe` if the location of the local variables and the input being the same would interfere with the result.

(todo: probably use some simpler opearator)

```
unsafe operator (int a % int b) -> int
    int res = a
    int t = 0
    int n = b
    int br = 1
    while(br != 0) do
        if(b > res)
            br = 0
        t = res >> 1
        n = b
        while (n < t) do
            n <<= 1
        if(br != 0)
            res -= n
    return res
```

## Types

This language has both ints and bools, and arrays are yet to be implemented completely.

(todo examples)

## Control Flow

This language has if statements, while loops and for loops.

(todo examples)

## Subroutines

(todo subroutines)
