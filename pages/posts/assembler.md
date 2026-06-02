---
title: Assembler Part One
date: 2026-02-07
description: writing an assembler
tag: C
author: Penguinanthan
---

### Compilers and interpreters are complicated

But assemblers are mostly not complicated. Therefore, in this series, we will be writing an simple assembler for a simple cpu: the SUBLEQ cpu.

What is subleq? Imagine trying to write an a x86 or an arm instruction set emulator. Both contain thousands of instructions, incredibly long opcodes, numerous addressing modes, numerous registers, andvariable length instructions. Such a project is a famously difficult undertaking that few have ever even bothered to try. However, the same project becomes 1000x simpler if instead the subleq instruction set is used. 

Subleq stands for "Subtract and branch if equal" and also happens to be, funnily, the only instruction in the perhaps mislabled instruction "set". Yes, that right: this is a turing complete cpu design that uses just one instruction to do literally everything. Stupidly simple. 

Despite being so ridiculously simple, this design is not unlike how microcoded cpu's already tend to work by offloading more complicated instructions onto "software" or more accurately, away from hardwired hardware. Where perhaps an instruction like LDA #10 (load an accumulator register with the immediate value 10) is broken down into a number of cpu signals in microcode rather than being a boolean logic equation synthesized into logic gates, an instruction like LDA #10 would be implemented using macros in the subleq assembler that consist soley of subleq instructions. It really is an ingenious little trick that perfectly mirrors how cpu's have evolved in the past 60 years or so. 

There's even a little Terry Davis-esc operating system written by one man using just the subleq ISA ([more here](https://https://esoteric.codes/blog/a-programming-language-with-only-one-command-and)). 

So how can we begin to implement the assembler for this? Well, first it might make more sense to get some sort of emulator up because while unit tests for the assembler can prove it's working, we need emulation to show we can get something more interesting than a fibbonaci program working. Most emulators (at least in c) follow a general state machine implemented as follows: The next instruction is fetched (usually indexing an array of 8 bit integers) and used to index an array of function pointers, one for each possible instruction in the ISA. In our emulator, though, this outline is made to literally be just fetch the next instruction and exectuing a function representing subleq that acts on memory. 

```
// One function for our only instruction
int32_t subleq() {
    int8_t addr_a = _cpu.memory[_cpu.pc];
    int8_t addr_b = _cpu.memory[_cpu.pc + 1];
    int8_t addr_c = _cpu.memory[_cpu.pc + 2];
    int8_t val_a = _cpu.memory[addr_a];
    int8_t val_b = _cpu.memory[addr_b];

    int8_t result = val_b - val_a;
    _cpu.memory[addr_b] = val_b - val_a;

    if (result <= 0) {
        _cpu.pc = addr_c;
    } else {
        _cpu.pc += 3;
    }

    return 1;
}

int32_t step() {
    // we'd normally index an array of function pointers here, but since it's subleq, we only need to call the function for that one instruction
	int8_t cycles = subleq();
	return cycles;
}

// our main api call to run the emulator for a specified number of cycles
void run(int32_t cycles) {
	while (_cpu.total_cycles < cycles) {
		_cpu.total_cycles += step();
	}
}
```

and that's it. The whole emulator is more or less 50-ish lines. 

Now we can move to the much harder part: the assembler. Assemblers, like compilers, can in a way be broken down as a series of transformations from one intermediate representation to another intermediate representation. The first IR is just the source code itself in all of our glorious custom syntax. Because this is merely an assembler, there are only two more IR's, so the pipeline would go something like this:

source code -> lexer -> parser -> encoder -> binary

and represented as some c structs here:

```
char *source;
```

-->

```
// first IR (tokens from a lexer)
typedef enum {
	MNEMONIC,
	DOLLAR,
	HASH,
	COLON,
	INT,
	SYMBOL,
	Eof,
	ILLEGAL
} Token_t;

typedef struct {
	char *literal;
	Token_t type;
} Token;
```
-- >
```
\\ Second IR (from parser)
typedef struct {
	// probbly just make these all subleq
	char *label;
	char *op_a;
	char *op_b;
	char *op_c;
	int address;
} ParsedInstruction;
```

--> encoded instructions (final binary)

