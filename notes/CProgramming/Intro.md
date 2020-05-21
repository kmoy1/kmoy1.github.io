# C: An Introduction

(*We're gonna assume y'all are using Windows or UNIX systems here. In the interest of space we won't cover Macs- but it's fairly easy to look up equivalent code or command snippets.*) 

Learning anything is best done by practice: programming in C is certainly no different. As is tradition, let's first print ```Hello World```. 

```C
#include <stdio.h>
int main(){
    printf("hello world\n");
}
```

Name this file ```hello.c``` or something. Given a C file, characterized by the ```.c``` extension, we must first **compile** it with the command ```gcc hello.c```:

This will generate an **executable file** named ```a.exe``` (Windows), while UNIX systems historically have generated ```a.out```. We can run executables by simply typing their name, in this case just ```a.exe``` or ```a.out```, and we run our compiled C program. 

Note if you're using a command shell like Bash, you can run any executable with ```./a.exe```. There's a reason for this that has to do with PATH environment variables and all that; don't worry about it. Other systems might have different rules too.

If you see ```hello world``` output from your terminal, congrats! You've created your first working C program!

(*We're gonna assume y'all are using Windows or UNIX systems from here on out. In the interest of space we won't cover Mac- but it's fairly easy to look up equivalent code or command snippets.*) 

## A C Program: Overview

C programs consists of **functions** and **variables**. Functions contain **statements**, single lines of code that do some operation. Variables, on the other hand, are like boxes that store values used during computation. 

In our program ```hello.c```, we had a function ```main```. Every program must have a function named ```main```, because it is the function where the program starts execution! In our program, ```main``` had a single statement ```printf("hello world\n");``` Notice also ```main ```encloses braces { } around its statements; this is done for all functions.

```main``` is where we call other functions, some that we wrote, and others from **libraries** already given to us. Libraries are files that we import that contain a bunch of functions we can use. For example, in ```hello.c```, notice the very first line contained ```#include <stdio.h>```, which tells our compiler (gcc) to include information about the *standard library*, which just contains a lot of useful input/output functions. We'll be likely using `stdio` a lot. 

A function has a list of values called **arguments** for a specific call. The parentheses after the function name surround the argument list. For example, if we called ```function(a,b,c)``` at some point in our code, our arguments are ```a,b,c```. Another example is ```main()```, a function that expects no arguments.

How do we *call* a function? Naming it, followed by a parenthesized list of arguments. So ```printf("hello world\n")``` names our function ```printf``` followed by a single argument, the text to print to output ```"hello world\n"```

Specifically, ```printf``` is a library function (stdio) that prints whatever was passed in as output.  A sequence of characters in double quotes ("hello, world\n") is called a **string**. 

The ```\n``` in the string is what is known as the *newline character*, which is basically the same as a single-spaced enter on your keyboard. We MUST specify this in order to make a new line of something printed *anywhere*: otherwise, you'd have to fit everything on a single line without ever returning.

There are other characters similar to ```\n``` : useful ones are ```\t``` for tab, ```\"```for the double quote and ```\\``` for a backslash character. Notice that using slash-character sequences that are invalid, like ```\c```, won't actually crash or error your program. 

## A More Practical Program

Let's make a program that makes a Fahrenheit-to-Celsius conversion table. 

```C
#include <stdio.h>
//Print table of Fahrenheit values converted to Celsius via formula C = (5/9) * (F - 32)
int main(){
    int f, c; //DECLARING variables before using them.
    int lower, upper, step;
    lower = 0; /* lower limit of temperature scale */
    upper = 300; /* upper limit */
    step = 20; /* step size */
    f = lower;
    while (f <= upper) {
        c = 5 * (f-32) / 9; //Celsius formula in code
        printf("%d\t%d\n", f, c);
        f = f + step;
    }
}
```

```//``` and ```/* ... */``` specify comments, which are ignored by the compiler. Commenting code is very important in general to others who might see or use your code to understand it. **Comments may appear anywhere where a blank, tab or newline can- i.e. anywhere not in the middle of code.**

In C, all variables must be **declared** before they are used, usually at the beginning (of the function). A declaration states the type of variable(s). ```int``` means integer. ```float``` means floating point (numbers like 3.14). Depending on if your machine is 32-bit or 64-bit, those specify the range of ```int``` and ```float```. There are also other types, like ```char, short, long,```and ```double```. There are multiple other types of objects, like arrays, structs, and unions, pointers to these objects, functions, etc. **Sizes** of all these objects are also machine-dependent. 	

### Temperature: Analysis

We first begin with assigning ```lower, upper,``` and ```step``` to some starting values. While loops repeat the same way, and print something each time. Here's how a while loop operates:

1. Evaluate condition. 
2. If true, execute loop body. Repeat step 1. 
3. If false, end loop, and continue execution at statement after loop.

Usually a while loop has braces to contain its loop body. However, if there's just one statement in this body, we can omit braces:

```C
while (i < j)
	i = 2 * i;
```

However, in either case, **indentation is important**. Loop body statements should be indented by a tab, which tells us these are loop body statements. C compilers don't actually care about this, but proper indentation and spacing are important for *readability*. Some other important readability guidelines:

- One statement per line
- Blanks around operators to clarify grouping, e.g. ```1 + 3``` 
- Brace positioning is definitely more subjective- pick one that fits you. The code examples above fit me. 

In ```temperature.c```, we compute a Celsius equivalent and assign it to ```c``` with ```c = 5 * (f-32) / 9;```. Why not multiply by 5/9? In C, as in many other languages like Java, **integer division truncates**: we always round down our answer, since we want our answer to maintain integer type. Since 5 and 9 are ints, 5/9 would be truncated to 0. This wouldn't be useful. 

```printf``` shows some more of its versatility here. It is a general-purpose output formatting function. Its first argument is a string of characters to be printed, where injected ```%(something)``` indicates some variable to be printed. For example, ```%d``` specifies an integer argument. Each```%(something)``` in the first string argument corresponds with the next argument to ```printf```. 

```printf``` is not part of C: C actually has no idea what "output" or "input" is. ```printf``` is just a useful function from a common library. However, the ANSI standard defines behavior of `printf`, so any compiler/library that follows ANSI standards will have ```printf``` functionality.

Another important C function is ```scanf```, now **reads** input into some variable instead of writing to output (terminal). 

One thing to notice: numbers on the right look a little "jolted"- they aren't right-justified. We can fix this by **augmenting each ```%d``` the ```printf``` call with a width**: for example, we might say ```printf("%3d %6d\n", f, c);``` instead of ```printf("%d %d\n", f, c)```to give each number field a given width. So now, output goes from

```
0       -17
20      -6
40      4
60      15
80      26
...
```

to 

```
  0    -17
 20     -6
 40      4
 60     15
 80     26
```

which looks more aligned (now right-justified) and thus nicer. 

## A Correction

Unfortunately, our code also contains actual mathematical error. Notice that all answers are integral, since we're doing integer operations. But this isn't right: $0^\circ F$ is around $-17.8^\circ C$, not $-17^\circ C$. **We can increase precision by using floats instead of ints.** Let's do this here real quick in ```temperature.c```:

```C
#include <stdio.h>
//Print table of Fahrenheit values converted to Celsius via formula C = (5/9) * (F - 32)
int main(){
    float f, c; //DECLARING variables before using them.
    float  	lower, upper, step;
    lower = 0; /* lower limit of temperature scale */
    upper = 300; /* upper limit */
    step = 20; /* step size */
    f = lower;
    while (f <= upper) {
        c = (5.0/9.0) * (f-32.0); //Celsius formula in code
        printf("%3.0f %6.1f\n", f, c);
        f = f + step;
    }
}
```

Note the changes we made here:

- Declared ```f,c``` to be ```float```.
- We can now use fractional multiplication because truncation (rounding down) no longer occurs.

Note: if we do arithmetic with a ```float``` and an ```int```,  the integer will be converted to a floating point *before* we carry the operation. So ```f-32``` and ```f-32.0``` are the same thing. Nevertheless, it's good practice to write floating-point constants with the decimal point for readability. 

Now let's look at ```printf```. The ```%3.0f``` in the format string says that a floating-point (```f```) is printed in a field *at least* 3 characters wide, no decimal point, no numbers after that decimal point (fraction digits). ```%6.1f``` describes another float (```c```) printed in a field at least *six* chars wide, 1 digit after decimal point. Thus output will look like:

```
  0  -17.8
 20   -6.7
 40    4.4
 60   15.6
 ...
```

So the number before the decimal point in ```%3.0f``` specifies *width*, and the number after specifies *precision*. Either can be omitted and simply has the effect of not enforcing the omitted restriction when printing.

```printf``` also can handle ```%o``` for octal, ```%x``` for hexadecimal, ```%c``` for character, ```%s``` for string arguments to format.  

## For Loops

Something we can do to make ```temperature.c``` even shorter: convert the while loop to a for loop!

```C
#include <stdio.h>
/* print Fahrenheit-Celsius table */
int main(){
    int f;
    for (f = 0; f <= 300; f = f + 20)
        printf("%3d %6.1f\n", f, (5.0/9.0)*(f-32));
}
```

Which seems *immensely* shorter than what we had before. What is this magic with a ```for``` loop, and what else did we do here to condense our code?

First, we eliminated most variables: only ```f``` (Fahrenheit) remains, and we made it back to an ```int```. We hardcoded upper limits and step size into the ```for``` loop. Finally, we stuck the statement calculating Celsius directly into the third argument of ```printf``` instead of variable ```c```. 

The last change we made is beautiful and allowed in almost every context and every language: we can use expressions that *return* a value in place of that actual value. 

The for loop is simply a generalization of the while loop, and occurs in 3 parts:

- **Initialization: **```f = 0```. This is done once *before* entering loop body.
- **Iteration**: Run through the loop body statements, utilizing that value of ```f``` where needed.
- **Increment**: ```f = f+20```. 
- **Check Condition**: ```f <= 300```. If true, repeat the iteration step. If false, exit the loop.

```while``` and ```for``` loops can generally be interchanged, but it's again a readability and scenario-dependent thing.

## Symbolic Constants

Finally, notice we hardcoded our lower and upper limits (0 and 300). This is generally bad practice. People might find readability issues later, and it's annoying to change if you want to make edits to the program. 

Instead of hardcoding, simply assign them to **symbolic constants**. ```#define```  defines a symbolic name or symbolic constant to be a particular string of characters: ```#define name replacement```. Any occurrence of name as a variable will then be immediately replaced by ```replacement```. The replacement text can be a string OR a number: type is implicitly defined. 

Let's see an example in ```temperature.c```:

```C
#include <stdio.h>
#define LOWER 0 //Symbolic constants defined before main()
#define UPPER 300 
#define STEP 20
/* print Fahrenheit-Celsius table */
int main(){
    int f;
    for (f = LOWER; f <= UPPER; f = f + STEP)
        printf("%3d %6.1f\n", f, (5.0/9.0)*(f-32));
}
```

```LOWER, UPPER, STEP``` are symbolic constants: we don't need to declare them in ```main()```. Convention for symbolic constants is full uppercase- so we know that it's a symbolic constant and not a variable name. Additionally, defining symbolic constants do not require semicolons at the end.

​	

