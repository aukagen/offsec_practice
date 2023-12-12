I am mad at myself for not having the writeup from someAssemblyRequired1, as I doo not remember how I did that..

(.wasm file --> web assembly, but in this I simply cat the file and got the flag which I inputted and was correct) I am very confused, as I dont understand where I got the wasm file from...
**UPDATE** I went to the ./<> which contained the file and looking at the contents I got this string `xakgK\Ns9=8:9l1?im8i<89?00>88k09=nj9kimnu`, which I knew from its position in the file that it was going to be the flag. Unfortunately, when I ran this in google there was a writeup that came up with how to decode it:
```
from pwn import *
print(xor(b"xakgK\Ns9=8:9l1?im8i<89?00>88k09=nj9kimnu", 8))
```
and so I got the flag. However, I wanted to learn how to do it proper + maybe even understand how other people did it so it was reading writeups time.

So my method appeared right, you dont really need to deobfuscate the js code and understand it THAT well, but it would hve showed that the encryption method was XOr with 8 bytes. 
Further, I saw someone use cyberchef's _magic_ option, which I had not come accross before but it is absolutely brilliant. 
![[Screenshot 2022-05-29 at 13.40.25.png]]

I did the same now, but instead of the flag being in clear text, it appears encoded:
![[Screenshot 2022-05-29 at 13.01.34.png]]

