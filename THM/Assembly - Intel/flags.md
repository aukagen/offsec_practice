**ZF** - Zero
--> set if the result is zero
**CF** - Carry
-> set if an _usigned_ arithmetic result is carried(add) or borrowed(sub)
**OF** - Overflow
--> set if _signed_ arithmetic result is too big for register
**SF** - Sign
--> set if a result is neg
**AF** - Adjust/Auxiliary
--> same as CF - for BCD operations (Binary Coded Decimal)
**PF** - Parity
--> set if num in last 8 bits = even
**TF** - Trap
--> for single stepping of programs
--> _(seem to remember this is useful when trying to figure out what a program does for exploits in pwn)_