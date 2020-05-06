- # Buffer Overflow Attacks


A *buffer overflow attack* is a storied and extremely common attack that spawns a *shellcode* for an attacker, which is basically gives them control over the machine. Yikes.

Let's say we were writing code that took user input somewhere. Note that this certainly doesn't restrict itself to direct input: this can be user input passed through multiple chains of function calls, or even through a user API. Once the user input is received, the program has to place it in a designated memory chunk called a **buffer**. If our input is *longer* than the space allocated to this buffer, and starts to extend into memory, we could potentially run into some problems!

This is mostly a problem with **lower-level languages** like C and C++, meaning these are languages have little abstraction, and that programmers can deal pretty much directly with memory itself. This means that unlike higher-level languages like Python and Java, there isn't any automatic error-checking for overflow! An infamous example occurs in C with its (*very much deprecated*) <code>strcpy</code> and <code>gets</code> functions, and the fact that it treats strings as byte sequences that only end with null terminator byte <code>0x00</code>. So C will gladly keep copying/reading bytes until it sees this <code>0x00</code>. 

### Program Memory Environment

Remember that *running programs have their own address space*, or a chunk of virtual memory for things like variables and code. This memory chunk is usually arranged into byte-sized slots. For example, on 32-bit systems, our address space would be <code>0x00000000</code> to <code>0xFFFFFFFF</code>. 

Going from low to high, our address space is split up into the following sections: 

- **Text**: Machine-level instructions for the program.
- **Data**: Global/static variables + other data known *at program creation*.
- **Heap**: Data whose size grows dynamically during program runtime. Grows upward.
- **Stack**: Variables and data local to the currently executing function (or *subroutine*). Grows downward in that calling a new function creates a new stack frame below the current one. So the "top" of the stack is really at the bottom, memory-address wise. 

In particular, the stack is incredibly important in understand how the buffer overflow exploit works. Typically, in a stack frame, we see more parts, this time from high to low memory address:

- **Return Address**: The address that points to the next line of code to execute after this subroutine returns. Often called the RIP, or return instruction pointer, for short.  
- **Arguments**: Arguments passed to this subroutine. 
- **Local Variables**: Local variables created during the running of this subroutine.





