## C: Intro, Part 2

Last note we gave an overview of a lot: how a C program compiled and ran, what the very basic components of a C program (and any program,for that matter) were. Let's extend upon our knowledge with data processing- like reading input.

## Character I/O

Let's now look at processing character data. We'll use the standard library (```stdio```) model of input and output, which is actually quite simple:

**All text input or output is treated as streams of characters.**

A **text stream** is a sequence of characters divided into lines; each line has some number of characters followed by a newline ```\n```. An Input stream, then, are just what you enter into the terminal and press return. The ```stdio``` library ensures each input or output stream follows this model. 

```stdio``` allows reading/writing one character at a time: ```getchar``` and ```putchar``` are the simplest. Each time it is called, ```getchar``` reads the next input character from a text stream and returns that character. Let's say we have statement ```c = getchar();``` ```c``` would contain the character read. The keyboard is normally the input vector for characters here, but there are definitely others as well, like files, which we'll discuss much later. 

```putchar```, logically then, prints a character. ```putchar(c);``` would print contents of ```c``` as a character, usually onto the screen.

### File Copying

Let's use ```getchar``` and ```putchar``` to write a program that copies input to output, one character at a time:

```C
#include <stdio.h>
/* copy input to output*/
int main(){
    int c;
    c = getchar();
    while (c != EOF) {
        putchar(c);
        c = getchar();
    }
}
```

Notice that we keep reading characters from input and printing them to output until we read the end of the input: i.e. end of file EOF. By the way, ```!=``` is the opposite of ```==```: it checks for inequality.

Internally (what the computer sees), characters and numbers are just bits (chunks of 0s and 1s). Why did we use ```int``` as our input type instead of ```char```? The reason is that ```getchar``` also needs to take in the EOF, which is an integer. A ```char```, which only holds a byte, is not big enough for this. Therefore we use ```int```, which can actually hold both chars (1 byte) and ints (4 bytes).

Assignment expressions actually return the value of the variable being assigned: thus, assignment can be part of a larger expression. We can combine the (re)assignment of ```c``` with our while loop like this:

```C
#include <stdio.h>
/* copy input to output; 2nd version */
int main(){
    int c;
    while ((c = getchar()) != EOF)
        putchar(c);
}
```

since the ```(c=getchar())``` statement returns character assigned to ```c``` as well. We now only need to reference ```getchar```once, and don't need to initialize it before the while loop. This we condense our code. Additionally, to people who understand this concept, this is easier to read. This style is common: however, we must also be careful to not get too carried away with this. 

Note we need the parentheses around ```c = getchar```. Comparison operators are higher than assignment, so precedence of ```!=``` >  ```=```, which means that in the absence of parentheses ```getchar() != EOF``` would be done before the assignment. This would have undesired effect of setting ```c``` to 1 or 0, corresponding to if ```getchar``` returned EOF or not, instead of a read character.

### Character Counting

Let's now draft a program to count characters:

```C
#include <stdio.h>
// count characters in input
int main(){
    long nc; //counter
    nc = 0; //initialize to 0
    while (getchar() != EOF)
        ++nc;
    printf("%ld\n", nc);
}
```

The statement ```++nc;``` introduces the **increment operator** ```++```; add by one. By itself, it's equivalent to ```nc = nc + 1```. The **decrement operator** ```--``` does the opposite: subtract by 1. The operators ++ and -- can be either **prefix** operators (++nc) or **postfix** operators (nc++): these become important when combining the increment operators with expressions, but we'll cover this later. 

For now, we'll use prefix operators to increment. Notice our character count is now stored in a ```long``` variable: longs are at least 32 bits, while ints are at least 16 bits. We'd want to do this in cases that integers might *overflow*: become so large it takes up more space than allocated. ```%ld``` tells ```printf``` that the corresponding argument is a long integer. 

A ```double``` (double precision float) allows for even bigger numbers than longs. Additionally, we can interchange between the above while loop and a for loop:

```C
#include <stdio.h>
/* count characters in input; 2nd version */
int main(){
    double nc;
    for (nc = 0; getchar() != EOF; ++nc)
        ;
    printf("%.0f\n", nc);
}
```

```printf``` uses ```%f``` for both float AND double. ```%.0f``` says to ignore the decimal points and anything after. Amazingly, our ```for``` loop is empty: all the counting work is done in the loop condition and increment! However, C requires ```for``` loop bodies have *something* : thus we place a lone semicolon, called a **null statement**.

One of the nice things about ```while``` and ```for``` loops is that a test is made **before** any iterations of body is executed. Good programs must account for all kinds of input- certainly including zero-length input. So for ```charcount.c```, our program would print 0, as it should, in this case, since our while/for condition immediately fails and our loop body (and thus increment) is never executed. 

### Line Counting

Now let's make ```linecount.c```, program to count input lines. The standard library defines an input text stream as a sequence of lines, each line terminated by a newline. So to count each line, we need to just count each newline character:

```C
#include <stdio.h>
// Count input lines
int main() {
    int c, nl; //remember why c is an int type
    nl = 0; //number of lines
    while ((c = getchar()) != EOF)
        if (c == '\n')
            ++nl;
    printf("%d\n", nl);
}
```

Note: be careful when using ```=``` (assignment) and ```==``` (equality Boolean). C compilers won't let you know. 

Notice that ```'\n'```, or any character in single quotes, is actually an integer value equal to a pre-assigned number (ASCII representation). These single-quoted characters are called **character constants**: they are the *exact same* as writing the assigned integer. So writing `'A'` would be the same as writing `65`, although for readability purposes you should of course use ```'A'```. ```'\n'``` is also 10 in ASCII- the assigned value for newline characters. NOTE: ```'\n'``` is a single character, and in expressions an integer; on the other hand, "\n" is a one-character string constant. 

### Word Counting

Now let's make ```wordcount.c```- counting words. Words are defined as any sequence of characters without tabs/blanks/newlines. 

```C
#include <stdio.h>
#define IN 1 /* inside a word */
#define OUT 0 /* outside a word */
/* count lines, words, and characters in input */
int main(){
    int c, nl, nw, nc, state;
    state = OUT;
    nl = nw = nc = 0;
    while ((c = getchar()) != EOF) {
        ++nc;
        if (c == '\n')
            ++nl;
        if (c == ' ' || c == '\n' || c = '\t')
            state = OUT;
        else if (state == OUT) {
            state = IN;
            ++nw;
        }
    }
    printf("%d %d %d\n", nl, nw, nc);
}
```

We can think of this program as a bunch of switches being flipped on and off when we are "in" a word and when we're not. Every time the program encounters the first character of a word, it increments word count ```nw```. The variable `state` lets us know at any time if our program has captured a character IN a word (IN) or not (OUT). IN and OUT (nice) are the same as 1 and 0- just given constants for readability and being easier to change later on.

The line `nl = nw = nc = 0;` sets all 3 variables to zero. This is not a special case: assignment is done right to left, and the result of assignment statements is simply the value assigned. An equivalent statement is    `nl = (nw = (nc = 0));`.

Expressions connected by `&&` or `||` are evaluated left to right, and evaluation stops immediately once the Boolean expression value is guaranteed: for example, if in `a || b || c`, if `a` is true, then `b` and `c`'s values won't matter. 

So now we've come up with a variety of input/output programs, just utilizing reading/writing single characters! Obviously, there's more functions that make this easier, but now we have a good feel for handling IO, which is of utmost importance in C. 