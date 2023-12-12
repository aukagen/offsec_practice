
![[Screenshot 2022-04-17 at 08.57.43.png]]![[Screenshot 2022-04-17 at 08.58.16.png]]
# Enumeration
## nmap
The scan showed SSH, FTP and HTTP (gunicorn)
It support the OPTION header - this could be useful
## dirb
![[Screenshot 2022-04-19 at 11.33.59.png]]
## curl
going to the web pages _/data/1-7_ does not give any results from the URL. CURLing them however showed some interesting information.
A couple users were found: _Ratul Hamba, Rashed, Kaji Patha_

_Amcharts, modernizr, colorlib_
_/download/1_ downloaded a wireshark file called 1. So I am assuming there might be more, or at least some information in this file. 


`zingchart.MODULESDIR = "https://cdn.zingchart.com/modules/";ZC.LICENSE = ["569d52cefae586f634c54f86dc99e6a9", "ee6b7db5b51705a13dc2339db3edaf6d"];` this was quite interesting? `
## wireshark
1.pkng showed nothing..
2.pkng had some data, but nothing useful just several _NOT FOUND_ error messages.
There were no more after 2, but after looking around there apparently was a 0.pkng:
the FTP stream showed an unencrypted login for _Nathan:Buck3tH4TF0RM3!_. Trying to FTP into this next.
Further, it looks like theres a notes.txt file..

## other
Colorlib template
http://10.10.10.245/?search=password# is the URL when trying t search for something, and I am sensing a vulnerability here.

# Foothold
Looking on the website it appears there are statistics for a user Nathan.
There are some statistics and overview of his network connections too.

There is a couple users 101(systemd-journal), 102(systemd-network) and 1001(nathan).

Theres a PID _8972/sh_ which could be interesting

## ssh login
using the credentials _Nathan:Buck3tH4TF0RM3!_ worked.
Looking around:
`ls` gives user flag
`sudo -l` did not work
some interesting files in the tmp directory..

Finding the notes.txt file: `find / -name notes.txt 2>/dev/null`
![[Screenshot 2022-04-19 at 12.05.26.png]]. In the tmp directcory there is a file _vmware-root_940-2689209484_, so if we can run this it will probably be as root?

_/usr/bin/gettext.sh_
_/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip_ --> this looks like can be used as SUID privesc?
## ftp login
same login worked for FTP.
# Privesc
running linpeas, a couple obvious CVE's showed:
![[Screenshot 2022-04-19 at 12.15.40.png]]

The baron Samedit looks like a heap based buffer overflow..
pwnkit takes advantage of setuid

could also be an exploit for the sudo version.. trying `sudoedit -s Y` as explained from [here](https://github.com/mohinparamasivam/Sudo-1.8.31-Root-Exploit) proved a deadend.

![[Screenshot 2022-04-19 at 12.19.45.png]]

I cloned [this](https://github.com/ly4k/PwnKit) into my tools directory, then proceeded to set up a python server and used _wget_ to download _PwnKit_ on the ssh machine of nathan. Then I simply ran `./PwnKit` and we got a root shell:))

# Flags
d1002ffa8dc404fdd7c2490f4fd2d8e8
8a42e1a566d21aee68b553ed6656257e
# Thoughts
Very easy, but first HTB box I have done COMPLETELLY without help so enjoyed it.
# New Things Learnt
- another privesc method would be to use the vulnerable python version (as mentioned): `python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'`
- `find / -perm -u=s -type f 2>/dev/null`
- `find / -perm -4000 -type f 2>/dev/null`