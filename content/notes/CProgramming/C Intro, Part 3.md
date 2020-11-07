# C: Intro, Part 3

Last note, we talked about using the standard library's treatment of IO streams and using `getchar` and `putchar` to read/write from these input/output streams. Let's move on to **arrays**.

## Arrays

An **array** is basically just a list of a specific type of value. For example, if you wanted to make something that held 10 integers, you'd just declare `int arr[10];` and use `arr[i]` to get the ith integer stored. 

Let's demonstrate with `allcount.c`, a program to count the number of occurrences of each character and digit in an input. 

```C
#include <stdio.h>
/* count digits, white space, others */
int main(){
    int c, i, nwhite, nother;
    int ndigit[10];
    nwhite = nother = 0;
    for (i = 0; i < 10; ++i)
        ndigit[i] = 0;
    while ((c = getchar()) != EOF)
        if (c >= '0' && c <= '9')
            ++ndigit[c-'0'];
    else if (c == ' ' || c == '\n' || c == '\t')
        ++nwhite;
    else
        ++nother;
    printf("digits =");
    for (i = 0; i < 10; ++i)
        printf(" %d", ndigit[i]);
    printf(", white space = %d, other = %d\n", nwhite, nother);
}
```

`int ndigit[10] ` declares `ndigit` an as a 10-int array. Arrays are **zero-indexed** in C: so the elements of `ndigit` are `ndigit[0]` to `ndigit[9]`. Note that subscripts (`[i]`) just has to be an int: expression or hardcoded.

This program relies on character representation of digits. For example, `if (c >= '0' && c <= '9')` tests if the character read into `c` is a digit. If it is, we can *convert* the `c` to its digit form by knowing there's an offset from the ASCII value: the numeric digit is `c - '0'`. Obviously, this offset calculation works only if the character digits have consecutive increasing ASCII values- which, fortunately, they do.

By definition, every `char` has an integer representation. This is natural and convenient; for example `c-'0'` is integer between 0 and 9 corresponding to the character '0' to '9' stored in `c`, and thus a valid index for `ndigit`. 

So classification of `c` into digits/whitespace/other is made by this code block: 

```C
while ((c = getchar()) != EOF)
    if (c >= '0' && c <= '9')
        ++ndigit[c-'0'];
else if (c == ' ' || c == '\n' || c == '\t')
    ++nwhite;
else
    ++nother;
```

## Functions

Let us now introduce the **function**, (in my opinion) the most important construct ever made in computer science. 

A function is basically a "package" of computation, which is just like a tool you can use without needing to worry about implementation (its code). With properly designed functions, we can implement **abstraction**: ignoring *how* a job is done and just knowing it's done correctly. 

So far, our only real usable functions (`main` doesn't really count) are `printf`, `getchar` and `putchar`. Those have been written for us already through libraries. Let's write some of our own. 

Let's write a function `power(m,n)` that returns the value of $m^n$, assuming $m$ and $n$ are integers. 

```C
int power(int m, int n){
    int i, p;
    p = 1;
    for (i = 1; i <= n; ++i)
        p = p * m;
    return p;
}
```

Basically we just multiply $m$ by itself $n$ times- that's exactly what $m^n$ is anyway.

So you can see that a function definition in C has form:

```
return-type function-name(parameters){
    declarations
    statements
}
```

Functions can appear in any order, and in one file or several, although **no function definition can be split between files**. We'll cover more on this later- this is not an issue of C but instead an operating system (compiler) issue.

The first line of the power function is sometimes referred to as the **signature**: `int power(int m, int n)`. We declare parameter types and names, and the return type. 

**Parameter names are local to a function**. They are not in scope anywhere else: other functions and main can use `m` and `n` without conflict. Additionally, variables defined inside functions (like `i` and `p` in `power`) are also local. 

Functions do not necessarily need to return a value; a return statement with no expression is specified by return type `void`. Such functions simply return control (point of code execution) to return to the caller (the line that called the void function). The calling function can ignore a value returned by a function. 

Why does `main` return an integer? The program **environment** (terminal) will receive this number and it will indicate if the program was a success or not. Return value `0` means a successful execution and termination. Any other values signal something else- either an error or a special case hit. By default, not returning anything for main returns `0` upon termination- program assumes successful run-thru. However, it's still good practice to specify `return 0;`, again to signal that program ends successfully when it hits this line. 

Let's look at incorporating `main` with `power`. 

```C
#include <stdio.h>
int power(int m, int n);
/* test power function */
int main(){
    int i;
    for (i = 0; i < 10; ++i)
        printf("%d %d %d\n", i, power(2,i), power(-3,i));
    return 0;
}
/* power: raise base to n-th power; n >= 0 */
int power(int m, int n){
    int i, p;
    p = 1;
    for (i = 1; i <= n; ++i)
        p = p * m;
    return p;
}

```

At the top before main, we see `int power(int m, int n);` This declaration is called a **function prototype**, and returns an error if the a function definition or any calls to that function do not agree with the prototype Parameter names don't need to match and are actually optional in a function prototype, so we could have just written `int power(int, int);` However, for readability once again, we include sample parameter names. 

## C is Pass-By-Value

In C, **all function arguments are passed by value**. This means function arguments are given in temporary variables instead of actual values. Pass-by-reference languages, on the other hand, gives functions access to original arguments instead of copies, which forces you to be careful with how you utilize those arguments in lieu of losing their original values permanently. 

Call-by-value is what makes `m` and `n` in `power` local to `power`. If we do something like decrement `n` in `power`, it won't actually decrement `n` outside of the function. 

However, C does let you "undo" this locality effect and modify the variable value. To do this, the caller must provide the **address** of the variable as the argument (a "**pointer**" to that variable), and function must declare a pointer parameter. Other than functions, pointers probably come in second as the most important computer science constructs ever made. We'll discuss much more about pointers later. 

For arrays, however, when we pass in an array (via its name),we pass in the pointer to the beginning of the array- we don't copy its elements. This way, the function can access its indexed values (and change them too!). So in this regard, functions with array parameters are pass-by-reference. 

## Character Arrays

Character arrays are extremely common in C. To demonstrate, let's write a program `longestLine.c` that reads a set of text lines and prints the longest. Here's some **pseudocode**:

```
while (there's another line)
	if (it's longer than the previous longest)
		(save it)
		(save its length)
print longest line
```

Here we implement **modularity**: a program divided into pieces that do parts in an assembly-line fashion. One function will read a new line, another will save it and its length. 

Let's write module 1: `getline` will read the next line of input. At minimum, `getline` must return a signal about possible end of file; return the length of the line, or zero if end of file is encountered. 

Now for module 2: `copy`will copy the new line to a safe place, for lines longer than the previous one saved. 

Finally, module 3: `main` controls `getline` and `copy`. Here is our final `longestLine.c`:

```C
#include <stdio.h>
#define MAXLINE 1000 /* maximum input line length */
int getline(char line[], int maxline);

void copy(char to[], char from[]);
//Print longest input line.
int main(){
    int len; /* current line length */
    int max; /* maximum length seen so far */
    char line[MAXLINE]; /* current input line */
    char longest[MAXLINE]; /* longest line saved here */
    max = 0;
    while ((len = getline(line, MAXLINE)) > 0)
        if (len > max) {
            max = len;
            copy(longest, line);
        }
    if (max > 0) /* there was a line */
        printf("%s", longest);
    return 0;
}
// getline: read a line into char array S and return length.
int getline(char s[],int lim){
    int c, i;
    for (i=0; i < lim-1 && (c=getchar())!=EOF && c!='\n'; ++i)
        s[i] = c;
    if (c == '\n') {
        s[i] = c;
        ++i;
    }
    s[i] = '\0';
    return i;
}
//COPY characters stored in TO into FROM, another buffer.
void copy(char to[], char from[]){
    int i;
    i = 0;
    while ((to[i] = from[i]) != '\0')
        ++i;
}
```

Note we have our standard function prototypes for `getline` and `copy`. 

Note we have our first example of a `void` function in `copy`: it doesn't return anything, but it does do something useful (stores stuff into `from`). 

Notice that `getline` also puts the **null terminator** `'\0'` (ASCII value zero) at the end of the array, which marks the end of a string. This conversion is also used by the C language: a string constant like "sentence\n"  is stored as an array of characters terminated with '\0':

<img src="C:\CProgramming\nullterminated.png" style="zoom: 33%;" />

`%s` in `printf` expects the corresponding argument to be a null-terminated string like above. `copy` **assumes** that its input char array `to` is null terminated as well to stop iteration.

We need to account for some memory issues here inherent in C. For example, what should `main` do on a line with length > `lim`? `getline` simply stops storing characters when the array is full.

There is no way for`getline` to know in advance an input line's length, so `getline` checks for **overflow**. However, `copy` assumes it already has safely passed-in strings of known length, so we choose to not to add error checking to it.

## External (Global) Variables and Scope

The variables we've seen declared in `main` so far are local- no other function can access/edit them. The same goes for any variables defined in functions. Each local variable disappears when the function exits.  These local variables are called **automatic**. 

Automatic variables do not retain their values between function calls. If they are not explicitly set, they will contain *garbage*. Alternatively, it is possible to define variables that are **external** (global) to all functions, i.e. variables accessible to *any* function. Global variables must remain in existence permanently, so they retain values even after the functions that set them return. Global variable definitions are done exactly once outside of functions. Every function that uses this global variable must also declare it, either explicitly with `extern` or implicitly. 

Let's demonstrate by rewriting `LongestLine.c` with `line`, `longest`, and `max` as external variables:

```C
#include <stdio.h>
#define MAXLINE 1000 /* maximum input line size */
int max; /* maximum length seen so far */
char line[MAXLINE]; /* current input line */
char longest[MAXLINE]; /* longest line saved here */
int getline(void);
void copy(void);
/* print longest input line; specialized version */
int main(){
    int len;
    extern int max;
    extern char longest[];
    max = 0;
    while ((len = getline()) > 0)
        if (len > max) {
            max = len;
            copy();
        }
    if (max > 0) /* there was a line */
        printf("%s", longest);
    return 0;
}
/* getline: specialized version */
int getline(void){
    int c, i;
    extern char line[];
    for (i = 0; i < MAXLINE - 1
         && (c=getchar)) != EOF && c != '\n'; ++i)
        line[i] = c;
    if (c == '\n') {
        line[i] = c;
        ++i;
    }
    line[i] = '\0';
    return i;
}
/* copy: specialized version */
void copy(void){
    int i;
    extern char line[], longest[];
    i = 0;
    while ((longest[i] = line[i]) != '\0')
        ++i;
}
```

The first few lines above `main`:

```C
int max; /* maximum length seen so far */
char line[MAXLINE]; /* current input line */
char longest[MAXLINE]; /* longest line saved here */
```

allocate storage for them. These definitions are *outside* of functions. Before a function can use an external variable, we must declare its name and type once again, and add `extern`. 

Actually, in this case, we actually don't need `extern` in the above code because the external variables were defined *before* it usage in the functions. In fact, placing all external variables at the beginning at minimizing use of `extern` is the standard. 

However, if a program spans multiple source files, and an external variable is defined one file and used in others, then `extern` is needed in the other files. Usually, external variable and function declarations go into a **header file**, which is #include-d in each source file that uses it. Header files have file extension `.h`. You might recognize one quickly: the standard library header file is included via `#include <stdio.h>`!  

By the way, we can don't need the `void` argument for `getline` and `copy` but include it anyway for compatibility with older C. We don't need to worry about that here. 

**Definition** of variables are where they are first created and allocated memory. **Declaration** of variables are just places where variable is stated but no memory allocated. 

In general, you have to be careful using global variables - they make the program harder to modify and global variables can change unexpectedly. Also, generality and modularity of program functions is sort of corrupted. 

## Conclusion: Intro

With this third note, our C intro concludes. Armed with the basic building blocks of C programs, we should now be able to write coherent, readable, and functional programs!

