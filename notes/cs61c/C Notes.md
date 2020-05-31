# CS 61C: C: Introduction, Pointers, & Arrays

History bullshit, history bullshit, etc. that I'm sure is incredibly important.

Any interpreter or compiler takes some defined computation encoding as part of input data (what it compiles/interprets). 

There is a parallel to hardware: instead of one machine per computation, we had the **stored program**: the computation was memory data, hardware (circuits) handled *interpretation* of that computation (carrying it out). This is the role of the CPU. 

C is a more or less low level language, where u really have to know how the hardware works- but the reward is you can do more with it and make shit faster. Less abstraction. Thus low-level languages are best for handling problems that deal with hardware itself- OS, compilers. 

**Overflow** occurs when computation produces an answer not representable in the computer's number representation.

*Conceptually* numbers are infinite, but expressing/representing those numbers is absolutely finite.

We can estimate powers of 2 by using $2^{10} \approx 1000$. 

Sometimes prefixes K and M (kilo, mega) mean 1024, 1024^2. To distinguish, we use KiBi for 1024, MeBi for 1024^2.

Modern PCs have 4 bytes (32 bits) for integers- integer arithmetic done entirely by hardware. Very fast. 

However, *portability* of C is an issue because word sizes might be different. This leads to subtle bugs. So Java specifies an int as always 4 bytes no matter what computer. So old 16-bit PCs need to use double precision ints, and 64-bit supercomputers need to truncate. 

Scheme integers are infinite in its size is only limited by the amount of memory available (unbounded-precision integer arithmetic). Scheme integers are also just a kind of number, NOT a representation format like most programming languages, so 1.0 is an integer in Scheme but not Java/C. 

## C is a terrible programming language

`*p++`increments the pointer *then* dereferences, since both operators have the same preference but we go right to left. So it's really `*(p++)`. However, not that because it's postfix, the value dereferenced is actually pointed to by `p`, not `p+1`. 

We cannot dereference anything that isn't a pointer- like an integer. So 

```C
int x, y;
y = *x;
```

is illegal.

However, if we *cast* integer `x` to a pointer first, *then* dereference, this is legal (but will still likely get a hardware error about fetching from a nonexistent address). So this is legal:

```C
int x, y;
y = * ((int *) x)
```

C was designed to allow writing of operating systems. One of the jobs of an OS is to control I/O devices. Processor communicates with I/O devices via **registers** with device status information, instructions for device tasks (from processor), etc. Processor views each register as part of memory. The processor reads and writes the registers just like actual memory.

Let's say device A had status register at address `0x1234`. Code:

```C
int addr,val; 
int *p;
addr = 0x1234; //Hardcode register address
val = *((int *)addr); //
p = ((int *)0x1234); //Pointer to address. 
val = *p; //Dereference pointer. 
```

If you treat an integer like a pointer as shown above, ensure that the integer is a valid memory address!

On declarations like `int i;`, compiler must allocate memory for variables. (Local variables alloc'd at runtime, global at compile time). 

A **declaration** only declares type for a variable. A **definition** declares type AND allocates memory. For example, a procedure declaration:

```C
int f(int x, char *p);
```

A procedure definition:

```C
int f(int x, char *p) {
    return x + strlen(p)
}
```

In C, a procedure must be declared before call. This will tell the compiler the expected arguments and return value, which lets the compiler fetch the appropriate machine instructions to carry execution. 

So a procedure *definition* allocates storage, namely the storage that contains the machine instructions to carry out the procedure. Note that this isn't necessary- you can just define the functions before using them in `main` so that there's no need for declarations- this is actually very common. Unfortunately, we can't do this in certain cases: for example, mutually recursive procedures can’t both be defined before each other. Another example: splitting a program into multiple files, so procedures are defined in one file and declared in another file (called a **header file**). 

