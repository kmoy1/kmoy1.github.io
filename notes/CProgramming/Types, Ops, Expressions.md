# Types, Operators, Expressions

We manipulate variables and constants in a program. Declarations create and name variables, specify their type, and maybe initialize them too. Operators do things, like add, these variables. Expressions combine variables and constants to produce a new value. The **type** of object will constrain its operations and possible values.

## Variable Names

There are some rules on how to name variables and symbolic constants: 

- Letters and digits only. 
- First character must be a letter (underscore `_` counts as a letter. Bad practice, though: library functions usually use these names) 
- lowercase letter $\neq$ uppercase letter. C standard: all lowercase for variables, all uppercase for symbolic constants
- No using keywords like `if, else, int`, etc. for variable names (in lowercase).

Choose variable names meaningfully and succinctly.

## Data Types and Sizes

- `char`: 1 byte
- `int`: integer size depending on architecture (4 bytes)
- `float`: single-precision float (don't worry what "precision" means right now- just remember this just allows decimal points)
- `double`: double-precision float

There's also `short` and `long`: `short` $\ge$ 2 bytes (16 bits), `long` $\ge$ 4 bytes (32 bits). `short` $\le$ `int` $\le$ `long`.

`signed` and `unsigned` qualifiers can be added to any `char` or `int`. `unsigned char` has value between 0 to 255, while `signed char` has value between -128 and 127 (for two's complement). Printable characters are always `unsigned` (positive).

## Constants

`ints` are straight numbers with no decimal points. `long` constants are written with terminal `l` or `L`: for example, `12345567890L`. Terminal `U` for unsigned, `UL` for `unsigned long`. Floating-point constants contain decimal point (`123.4`), exponent (`1e-2`) or both; type is `double` unless suffixed. Terminal `f ` or `F` = float constant; `l` or `L` = long double.

We can write ints in octal or hexadecimal too. Leading `0` = octal,  leading `0x` or `0X` means hexadecimal. Example: `31` can be written as `037` in octal, `0x1f` or `0x1F` in hex. Since they are still ints, we can still terminate these formats with `L` or `U`.

Character constants denoted like `'x'` are still integers equal to the numeric value of the character in the machine's character set (ASCII). We just write out the character for readability.

Character constant `'\0'` is the null character, with value 0. Writing `'\0'` and `0` are identical- again we write the former for readability purposes, specifying a character nature. 

A **constant expression** is an expression with only constants. These expressions might be evaluated at compile time (rather than runtime). For example, `1 + 3 + 4 + 4` is a constant expression.

**String constants or string literals** are double-quoted sequences of characters. The *quotes are not part of the string*: they are mere delimiters. `\"` is the double-quote character. At compile time, string constants are *concatenated*: `"I am" " an apple"` concatenates to `"I am an apple"`. If we have a really long string over multiple times, this will help.

A string constant is a *null-terminated array of characters*. We have `'\0'` at the end of the array, we need take number of characters + 1 space. Thus there's no limit to string length, but we have to scan through a string completely (until we hit the null terminator) to find length. `<string.h>` has function `strlen(s)`to return the length of string `s` (excluding `'\0'`). We can actually just implement this real quick:

```C
int strlen(char s[]){
    int i;
    while (s[i] != '\0')
        ++i;
    return i;
}
```

Characters are NOT strings: `'x'` $\neq~$ `"x"`. `x` is an integer, while `"x"` is an array of characters with character `'x'` and null terminator `'\0'`.

An **enumeration** is a list of constant integers. For example: 

```C
enum boolean { NO, YES };
```

We can specify the values of the enum:

```C
enum escapes { BELL = '\a', BACKSPACE = '\b', TAB = '\t',
              NEWLINE = '\n', VTAB = '\v', RETURN = '\r' };
```

By default, values increment progressively from left to right of the list- starting at 0 if no values are specified. 

We use enumerations to give meaningful names to constants: an alternative to `#define`. We don't really need to specify type for enums, although we can. A debugger can also map our constants to its name, making life easier. 

## Declarations

A declaration specifies a type, and contains a list of one or more variables of that type. For example:

```C
int lower, upper, step;
char c, line[1000]; 
```

Variables can also be initialized in declaration:

```C
char esc = '\\';
int i = 0;
int limit = MAXLINE+1;
float eps = 1.0e-5;
```

Except for `automatic` variables (don't worry if you don't know what these are), initialization is done once *before* program execution. Initial value must be a constant expression. `external` and `static` variables zero-initialized by default. Uninitialized variables have garbage values.

`const` can be added to any variable declaration to specify a constant value- cannot be changed throughout the program. `const` for arrays states that elements cannot be changed. For example:

```C
const double pi = 3.1415926;
const char msg[] = "A permanent message.";
```

`const` can also be used with array arguments, to prevent a function from changing its elements: `int strlen(const char[]);`

## Arithmetic Operators

Binary arithmetic operators: `+, -, *, /, %`. Integer division truncates. `x % y` produces the *remainder* of x / y . The `%` operator can only operate on integers. If operands are negative integers, then the result is machine-dependent.

`+` and `-` have the same precedence, lower than the precedence of `*, /, %`.  All are lower than **unary** operators `+` and `-`.  Arithmetic operators apply left to right. 

## Relational and Logical Operators

Relational operators: `< , >, <=, >=`, all with same precedence. Equality operators `==` and `!=` have precedence one level below. **Relational operators have lower precedence than arithmetic operators**: `i < x+y` is the same as `i < (x+y)`.

### Logical Operators

Logical operators: `&&` and `||`. Like arithmetic operators, logical operators apply left to right, and evaluations stops immediately upon finding an overall true/false value. Let's examine the for loop in our `getline` function from a few notes ago, which read an input line into a character array `s`:

```C
for (i=0; i < lim-1 && (c=getchar()) != '\n' && c != EOF; ++i)
    s[i] = c;
```

We need to check if there's room in `s` to store the array *first*; the test `i < lim-1` is thus first. If this test fails, `getchar()` and the `EOF` comparison does not execute (as it shouldn't).

 `&&` has higher precedence than `||`:  both are lower than relational and equality operators. But `!=` has higher precedence than assignment (`=`) so we need to specify parentheses in `(c=getchar()) != '\n'`  in order to assign to `c` *before* comparing to `'\n'`.

Logical or relational expressions are also called **Boolean expressions**, and have numeric value 1 if true, 0 if false. 

The **negation operator** `!` negates true to false, false to true. It's an alternative to comparing to 0: for example, we can now use `if (!valid)` instead of `if (valid == 0)`. 

## Type Conversion

