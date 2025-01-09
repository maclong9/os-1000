# OS in 1,000 Lines

Notes from following along with [this book](https://operating-system-in-1000-lines.vercel.app/en/02-assembly). This project utilises the C programming language to write a basic 32-bit Operating System for RISC-V processors and QEMU virtual machine, this is so you can emulate a RISC-V processor rather than having to purchase physical hardware. 

## RISC-V Assembly 101

RISC-V, or RISC-V ISA (Instruction Set Architecture), defines the instructions that the CPU can execute. It's similar to an API or programming language specification. When you write a C program, the compiler translates it into RISC-V assembly.

> [!NOTE]
> You will need to write some Assembly code when writing an OS, however it isn't as difficult as you might think. 

> [!TIP] 
> Using [Compiler Explorer](https://godbolt.org/) to see how higher level languages compile down to Assembly is a great idea!


