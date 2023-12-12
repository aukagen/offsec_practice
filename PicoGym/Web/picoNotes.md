picoCTF{th4ts_4_l0t_0f_pl4c3s_2_lO0k

More Cookies
===
[homomorphic encryption](https://spring.io/blog/2014/01/20/exploiting-encrypted-cookies-for-fun-and-profit) + [this medium article](https://medium.com/privacy-preserving-natural-language-processing/homomorphic-encryption-for-beginners-a-practical-guide-part-1-b8f26d03a98a)

I found [this](https://crypto.stackexchange.com/questions/66085/bit-flipping-attack-on-cbc-mode) aticle which had some interesting code:
```python
def bitFlip( pos, bit, data):
    raw = b64decode(data)

    list1 = list(raw)
    list1[pos] = chr(ord(list1[pos])^bit)
    raw = ''.join(list1)
    return b64encode(raw)
```
and it looks like itll be a CBC bit-flip attack. However, this code is not enough..
Functional code (only when using Python2):
```python
#!/usr/bin/env python

from base64 import b64decode
from base64 import b64encode
import requests


# script from Martin Carlisle
s = requests.Session()
s.get('http://mercury.picoctf.net:34962/')
cookies=s.cookies["auth_name"]
#print(cookies) # works

unb64 = b64decode(cookies)
unb64 = b64decode(unb64)
#print(unb64) # works - simply returnes the raw string


for i in range(128):
	pos=i//8 # each position is 8 bytes long
	guessDec = unb64[0:pos]+chr(ord(unb64[pos])^(1<<(i%8)))+unb64[pos+1:]
	guessEnc1 = b64encode(guessDec)
	guess = b64encode(guessEnc1)
	
	r = requests.get('http://mercury.picoctf.net:34962/',cookies={"auth_name":guess})
	
	if 'picoCTF{' in r.text:
		print(r.text)
```
flag: 	![[Screenshot 2022-05-23 at 12.24.57.png]]


ItsMyBday
===
You would get th flag if you upload 2 _different_ files with the _same_ md5 hash. I was not sure how to do this, but researching I found an [article](https://natmchugh.blogspot.com/2014/10/how-i-made-two-php-files-with-same-md5.html?m=1). I downloaded the 2 files and got the flag, but also wanted to learn how to do it properly!
--> this would be done with the fastcoll tool (downloaded but oculdn't compile)


