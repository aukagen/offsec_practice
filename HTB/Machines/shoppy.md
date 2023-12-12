
running `gobuster vhost -u shoppy.htb -w Tools/SecLists/Discovery/DNS/dns-Jhaddix.txt --no-progress -t 50 --no-error -o gobuster_result` I got a 200ok on a mattermost.shoppy.htb website!
--> this also has a login page, but after attempting the same nosqli it did not work. 

It seem spossible to attempt logging in without filling in one of the form fields (username/password)

I just KNOW its sqli

 performing a second nmap scan I found a copycat service running on port 9093, and after a quick search it appears it is a copycat database replication service

I also think there will be a subdomain called beta, with the hints on the homepage.
going to support.htb:9093 looks like there is a playbook with version 1.29.1
appears it might be a GoLang sqli?

 trying _admin' or 1=1 -- -_ mkes the page hang and then go to a 504 error page. This does not happen when I use double quotes so there should be an sql injection in the go db I think
go_info{version="go1.18.1"} 

 by using gobuster with the vhost option, I got a subdomain called mattermost with a 200 OK code. Navigating to this page there is another login page. It appears that mattermost is a fframework  with loads of plugins such as wordpress so I am hoping there will be some comomn vulnerabilities

trying to brute navigate to <target>/api, I am redirected to the login page but with a _?redirect_to=%2Fapi_ at the end of the URI --> possibility for path ttraversal or something?
Further, the httpOnly flag is not set on ANY of the large amount of cookies. It also appears the same cookies are being used for both of the domains, so I believe if I can get authenticated on one I can use that for the other...

According to https://nullsweep.com/nosql-injection-cheatsheet/ it appears that if any of the special characters makes the page return a 500 error it is likely vulnerable to NoSQLi --> as mentioned earlier a 504 error is returned when I use the ', so I am going to try to get authenticated using NoSQLi

I attempted payloads similar to: _' || 'a'=='a_ but it did not work. It was not until I swapped out the a's with 1's that authentication bypass worked

admin' || '2==2 worked

when I logged in and went intoinspect mode on the search page a 'download exports' button popped up. It appeared it was in the /exports/ directory, so I saved the usernames(admin, john), ids(62db0e93d6d6a999a66ee67a, 62db0e93d6d6a999a66ee67b) and pwd_hashes(23c6877d9e2b564ef8b32c3a23de27b2, 6ebcea65320589ca4f2f1ce039975995)

NEXT:
- perhaps try crcacking the hashes (md5 so will take long) --> definitely try password bruteforcing with ssh and hydra + use the cookies from shoppy.htb being logged in to log in for the mattermost page!! Also, try to use ffuf to get /exports/*.json files:))


For the username parameter in exports I entered the same payload as I used to bypass authentication which gave me the details above^

putting both of the hashes into a file and using `hashcat -a 0 -m 0 hashes /usr/share/wordlists/rockyou.txt` one of the hashes is cracked: 6ebcea65320589ca4f2f1ce039975995:remembermethisway
\In the channel there were 2 other users: jaeger and jess


Looking around on the subdomain I find our Jess has a cat Tigrou whom she normally brings to work in August --> so her password might be something along those lines. Josh has deployed a password manager built with C++ to the deploy machine --> hopefully there are some passwords to be found there...

using the credentials jaeger:Sh0ppyBest@pp! I got ssh access and foothold + the user flag: e5dae491773e0ebc5c5287dce70aa191. 
** WHERE DID YOU GET THE PASSWORD??**
Looking at the other files there is an executable shoppy_start.sh. I am going to play with that for a short while before I upload linpeas and go from there

Looking at the path: /home/jaeger/.nvm/versions/node/v18.6.0/bin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games there might perhaps be a path poisoning privesc

Aim: run the password-manager binary, as this will read the contents of creds.txt. Only catch is to find josh's master password (unless he has reused his previous one)d
.Analysing the file with `xxd /home/depoy/password-manager`, the password is in the code: Sample. This gives the creds _deploy:Deploying@pp!_ which I am not 100% sure what to do with but I assume ssh?

UPDATE: sshing into the machine with these credentials gave me another shell, but it looks to be even less privileged...
 However, with this deploy user I am in a docker group, and I think that can be leveraged for root...
 
 
# Privesc:
SSHin in as the deploy user, I am part of the docker group. After checking for files this group has access to, I come accross the /run/docker.sock. Looking it up I find an [article](https://book.hacktricks.xyz/linux-hardening/privilege-escalation#writable-docker-socket) on hacktricks which explains that if I can write to it, I can run a privileged docker instance with the following command:`docker -H unix:///var/run/docker.sock run -v /:/host -it ubuntu chroot /host /bin/bash`. 
Trying put that exact command I got an 'no ubuntu:latest' running locally. So I did a quick check with `image ls -a` and it turned out there was an alpine image running. From previous experience, alpine has been used for many privescs, so I tried the same command but exchanged ubuntu with alpine and got a root shell + the flag.

Thoughts:
The ernumeration part to first bypass the user login was what took me the longest and I had to look for some help with it. Not being that familiar with NoSQLi and also not being super proficient in SQLi I found it silghtly confusing. However, once I got past that bit it was all very straight forward:)
