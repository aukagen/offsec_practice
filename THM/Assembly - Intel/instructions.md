Immediate - e.g. 12 (const data)
Register - eg RAX, EAX, AL, R8
Memory (Address) - e.g. 0x77480938

Format
===
_(Instructions/Opcode/Mnemonics) Destination_Operand, Source_Operand_

Common
===
### Moving Data
**MOV** - moving/storing (src operand in destination)
**LEA** - Load Effective Address (computes addresses) ADDRESSES ONLY
**PUSH** - putting something on top of stack (acts as copying so register/value does not change)
**POP** - takes whats on top of the stack and stores it in given destination

### Arithmetic
**INC** - increments data by 1
**DEC** - decrements a value by 1
**ADD** - adds a source to a destination and stores the result in destination
**SUB** - substracts a source from a destination and stores the result in destination

### Multiplication + Division
- first a value must be stored in RAX, then one in RBX. The arithmetic is then performed by calling it and RBX --> the arithmetic is then applied to the result in RAX with the value in RBX --> then stored in RDX:RAX (but referencing RAX should be enough)
**MUL**/**IMUL** - multiplication (unsigned/signed)
**DIV**/**IDIV** - division (unsigned/signed)

### Flow Control
**RET** - return (used with RAX to return values)
**CMP** - compare (2 operands - sets flag according to result)
**J'CC'** - conditional jump (based on flags currently set. CC is not valid --> JNE, JLE, JNZ, JG, etc..) _Not Equal_, _Less than or Equal_, _if flag is Not Zero_, _Greater than_ aka 'if statements'
**NOP** - No Operation (for padding)

### Pointers
"Dereferencing to get values from inside memory addresses"
--> essentially a complicated way of doing more basic instructions
	LEA[2] + Square Brackets[1]:
	1. Refers to accessing the _Memory Address_ of a [var] not the [var]/value  itself
	2. calculating + loading addresses
**When working with LEA, square brackets do NOT dereference**:
![[Screenshot 2022-06-04 at 11.28.40.png]]

### Zero Extension
`movzx` --> for always zero extending

### JMP
Unsigned Comparisons:
```
    JB/JNAE (CF = 1) ; Jump if below/not above or equal

    JAE/JNB (CF = 0) ; Jump if above or equal/not below

    JBE/JNA (CF = 1 or ZF = 1) ; Jump if below or equal/not above

    JA/JNBE (CF = 0 and ZF = 0); Jump if above/not below or equal
```

Signed Comparisons:
```
	JL/JNGE (SF <> OF) ; Jump if less/not greater or equal

	JGE/JNL (SF = OF) ; Jump if greater or equal/not less

	JLE/JNG (ZF = 1 or SF <> OF); Jump if less or equal/not greater

	JG/JNLE (ZF = 0 and SF = OF); Jump if greater/not less or equal
```