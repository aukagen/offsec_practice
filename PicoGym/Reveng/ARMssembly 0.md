ARMssembly0
===

### Running the program
(this is the method to run any x86 armv8 binary on a non-x64 system!)
1. `aarch64-linux-gnu-as -o chall.o chall.S`
2. `aarch64-linux-gnu-gcc -static -o chall chall.o`
3. `./chall 266134863 1592237099`
I used [this](https://github.com/joebobmiles/ARMv8ViaLinuxCommandline) and [this](https://azeria-labs.com/arm-on-x86-qemu-user/) to help install th required packages.
The output I then got was ![[Screenshot 2022-05-29 at 10.00.44.png]], which I then simply had to convert to hex!
Note to future self: assembly can be confusin, rather than try to read what it does, execute the code;)
_[source](https://picoctf2021.haydenhousen.com/reverse-engineering/armssembly-0)_


![[Screenshot 2022-05-28 at 12.11.14.png]]

a good source on [ARM64]https://cit.dixie.edu/cs/2810/arm64-assembly.html
Other good resources:
[ARM instruction set](https://azeria-labs.com/arm-instruction-set-part-3/)
[architecture ref man](https://developer.arm.com/documentation/ddi0487/latest)
assembly language used is x86 assembly - in ARM structure


sp = stack pointer
sub = substraction
str = store task register
ldr = load register
ldp = load framepointer
cmp = compare 2 values
bls [x, y] = branch if x less than y
b [ne/eq/..]= normal branch
add = addition
ret = return from a function using value of previous bl
bl (atoi) = branch-and-link instruction --> atoi is the function (ascii to int). It puts a return address in x30 and is the _normal_ way of calling a function
mov = copies contents from one location to another
[adrp](https://stackoverflow.com/questions/41906688/what-are-the-semantics-of-adrp-and-adrl-instructions-in-arm-assembly) = shifts pages


stp = part of standard start of a non-leaf function:
```
stp    x29, x30, [sp, #-16]!
mov    x29, sp
``` 
--> stores frame pointer (x29) and link register (x30) onto the stack, and the memory location for this is computed using sp -_minus_ 16 --> which updates the sp address and makes it the same as the fp.
```
stp    x29, x30, [sp, #-48]!
add    x29, sp, 0
```
--> ^this was the code from the challenge, but essentially does the same (with -48 in place of 16)


.size =
.section = 
.align = 
.type _funcName_ %function = runs the function

**ESSENTIALLY, what it did was compare the 2 inputs, and then converted the longer one from the integers to hex**