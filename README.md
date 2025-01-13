# OS in 1,000 Lines

**Notes from following along with [this book](https://operating-system-in-1000-lines.vercel.app/en/02-assembly).** 

This project utilises the C programming language to write a basic 32-bit Operating System for RISC-V processors and QEMU virtual machine, this is so you can emulate a RISC-V processor rather than having to purchase physical hardware. 

## RISC-V Assembly 101

RISC-V, or RISC-V ISA (Instruction Set Architecture), defines the instructions that the CPU can execute. It's similar to an API or programming language specification. When you write a C program, the compiler translates it into RISC-V assembly.

> [!NOTE]
> You will need to write some Assembly code when writing an OS, however it isn't as difficult as you might think. 

> [!TIP] 
> Using [Compiler Explorer](https://godbolt.org/) to see how higher level languages compile down to Assembly is a great idea!

### Assembly Language Basics

Assembly is a mostly direct representation of machine code.

``` asm
addi a0, a1, 123
```

Typically each line of assembbly code represents a single instruction. The first column is the instruction name, also known as the _opcode_. The following columns are the _operands_, the arguments for the instruction, In this case the `addi` instruction adds the value `123` to the value in register `a1`, and stores the result in register `a0`.

### Registers

Registers are like temporary variables in the CPU, they are much faster than memory. CPU reads data from memory into registers, does arithmetic operations on registers and then writes the results back to memory/registers. 

> [!NOTE]
> Registers are stored inside of the CPU, each core typically has its own set of registers. This allows each core to operate independently and perform calculations without intefering with others or having to fetch the data from cache or memory.

**Common Registers in RISC-V**

| Register    | ABI Name (alias) | Description                                         |
|-------------|------------------|-----------------------------------------------------|
| `pc`        | `pc`             | Program counter (where the next instruction is)     |
| `x0 `       | `zero`           | Hardwired zero                                      |
| `x1`        | `ra`             | Return address                                      |
| `x2`        | `sp`             | Stack pointer                                       |
| `x3 - x4`   | `gp - tp`        | Global pointer - Thread pointer                     |
| `x5 - x7`   | `t0 - t2`        | Temporary registers                                 |
| `x8`        | `s0/fp`          | Stack frame pointer                                 |
| `x9`        | `s1`             | Frame pointer                                       |
| `x10 - x11` | `a0 - a1`        | Function arguments/return values                    |
| `x12 - x17` | `a2 - a7`        | Function arguments                                  |
| `x18 - x27` | `s2 - s11`       | Temporary registers saved across calls              |
| `x28 - x31` | `t3 - t6`        | Temporary registers                                 |

> [!NOTE]
> The Apple M-Series CPUs are based on ARM architecture, which typically features 31 general-purpose registers per core.

> [!TIP]
> You may use CPU registers as you like, but for the sake of interoperability with other software, how registers are used is well defined. For example, `x10` – `x11` registers are used for function arguments and return values.

### Memory Access

Registers are incredibly fast, yet limited in number. Most data is stored in memory, and programs read/write data from/to memory using the `lw` (load word) and `sw` (store word) instructions: 

``` asm
lw a0, (a1)  // Read a word (32-bits) from address in a1
             // and store it in a0. In C, this would be: a0 = *a1;
```

``` asm
sw a0, (a1)  // Store a word in a0 to the address in a1.
             // In C, this would be: *a1 = a0;
```

You can consider `(...)` as a pointer dereference in C. In this case, `a1` is a pointer to a 32-bits-wide value.

### Branch Instructions

Branch instruction change the control flow of the program. They are used to implement, `if`, `for`, and `while` statements: 

``` asm
    bnez    a0, <label>   // Go to <label> if a0 is not zero
    // If a0 is zero, continue here

<label>:
    // If a0 is not zero, continue here
```

`bnez` stands for "branch if not equal to zero". Other common branch instructions include `beq` (branch if equal) and `blt` (branch if less than). They are similar to `goto` in C, but with conditions.

### Function Calls

`jal` (jump and link) and `ret` (return) instructions are used to call functions and return from them:

``` asm
    li  a0, 123      // Load 123 to a0 register (function argument)
    jal ra, <label>  // Jump to <label> and store the return address
                     // in the ra register.

    // After the function call, continue here...

// int func(int a) {
//   a += 1;
//   return a;
// }
<label>:
    addi a0, a0, 1    // Increment a0 (first argument) by 1

    ret               // Return to the address stored in ra.
                      // a0 register has the return value.
```

Function arguments are passed in  `a0`–`a7` registers, and the return value is stored in `a0` register, as per the calling convention.

### Stack

Stack is a Last-In-First-Our (LIFO) memory space used for function calls and local variables. It grows downwards, and the stack pointer `sp` points to the top of the stack. 

To save a value to the stack, decrement the stack pointer and store the value:

``` asm
    addi sp, sp, -4  // Move the stack pointer down by 4 bytes
    sw   a0, (sp)    // Store a0 register to the stack
```

To load a value from the stack, load the value and increment the stack pointer:

``` asm
    lw   a0, (sp)    // Load a0 register from the stack
    addi sp, sp, 4   // Move the stack pointer up by 4 bytes
```

> [!TIP]
> For simplicity these operations are sometimes called `push` and `pop` because you `push` a new item into the stack and `pop` the top item off the stack.


