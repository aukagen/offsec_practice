id = 18
username = desktop-qv3nqlm\larry stevens
system = windows
![[Screenshot 2022-07-16 at 10.17.51.png]]
![[Screenshot 2022-07-16 at 10.18.08.png]]


id = 19
```bash
Interface: 192.168.137.131 --- 0x6
  Internet Address      Physical Address      Type
  192.168.137.1         00-50-56-c0-00-01     dynamic   
  192.168.137.254       00-50-56-f8-3b-90     dynamic   
  192.168.137.255       ff-ff-ff-ff-ff-ff     static    
  224.0.0.22            01-00-5e-00-00-16     static    
  224.0.0.251           01-00-5e-00-00-fb     static    
  224.0.0.252           01-00-5e-00-00-fc     static    
  239.255.255.250       01-00-5e-7f-ff-fa     static    
  255.255.255.255       ff-ff-ff-ff-ff-ff     static    
  ```

id = 20
```
   Host Name . . . . . . . . . . . . : DESKTOP-QV3NQLM
   Primary Dns Suffix  . . . . . . . : 
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No
   DNS Suffix Search List. . . . . . : localdomain

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : localdomain
   Description . . . . . . . . . . . : Intel(R) 82574L Gigabit Network Connection
   Physical Address. . . . . . . . . : 00-0C-29-54-D1-53
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
   Link-local IPv6 Address . . . . . : fe80::6067:1152:ba6c:26c7%6(Preferred) 
   IPv4 Address. . . . . . . . . . . : 192.168.137.131(Preferred) 
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Lease Obtained. . . . . . . . . . : 24 June 2022 17:48:13
   Lease Expires . . . . . . . . . . : 24 June 2022 22:33:14
   Default Gateway . . . . . . . . . : 
   DHCP Server . . . . . . . . . . . : 192.168.137.254
   DHCPv6 IAID . . . . . . . . . . . : 100666409
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-29-38-3A-58-00-0C-29-54-D1-53
   DNS Servers . . . . . . . . . . . : 192.168.137.1
   NetBIOS over Tcpip. . . . . . . . : Enabled
```


werkzeug 2.1.2
PostgreSQL
![[Screenshot 2022-07-17 at 08.06.30.png]]
![[Screenshot 2022-07-17 at 08.06.55.png]]

Master Browser Server Name: THUNDERPALACE2


id = 21
```
System error 1231 has occurred.

The network location cannot be reached. For information about network troubleshooting, see Windows Help.
```

id = 22
```bash
'set"' is not recognized as an internal or external command,
operable program or batch file.
```

id = 23
```bash
*** Request to UnKnown timed-out

DNS request timed out.
    timeout was 10 seconds.
Server:  UnKnown
Address:  192.168.137.1

DNS request timed out.
    timeout was 10 seconds.
DNS request timed out.
    timeout was 10 seconds.
```


id = 24
```bash
Enumerating domain trusts failed: Status = 1722 0x6ba RPC_S_SERVER_UNAVAILABLE
```


id = 25
```bash

Share name   Resource                        Remark

-------------------------------------------------------------------------------
C$           C:\                             Default share                     
IPC$                                         Remote IPC                        
ADMIN$       C:\Windows                      Remote Admin                      
The command completed successfully.

```

id = 26
```bash
===========================================================================
Interface List
  6...00 0c 29 54 d1 53 ......Intel(R) 82574L Gigabit Network Connection
  1...........................Software Loopback Interface 1
===========================================================================

IPv4 Route Table
===========================================================================
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
        127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
        127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
  127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
    192.168.137.0    255.255.255.0         On-link   192.168.137.131    281
  192.168.137.131  255.255.255.255         On-link   192.168.137.131    281
  192.168.137.255  255.255.255.255         On-link   192.168.137.131    281
        224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
        224.0.0.0        240.0.0.0         On-link   192.168.137.131    281
  255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
  255.255.255.255  255.255.255.255         On-link   192.168.137.131    281
===========================================================================
Persistent Routes:
  None

IPv6 Route Table
===========================================================================
Active Routes:
 If Metric Network Destination      Gateway
  1    331 ::1/128                  On-link
  6    281 fe80::/64                On-link
  6    281 fe80::6067:1152:ba6c:26c7/128
                                    On-link
  1    331 ff00::/8                 On-link
  6    281 ff00::/8                 On-link
===========================================================================
Persistent Routes:
  None
```



id = 27
```bash

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       916
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:5040           0.0.0.0:0              LISTENING       1120
  TCP    0.0.0.0:5357           0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING       704
  TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING       540
  TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING       552
  TCP    0.0.0.0:49667          0.0.0.0:0              LISTENING       460
  TCP    0.0.0.0:49668          0.0.0.0:0              LISTENING       1772
  TCP    0.0.0.0:49669          0.0.0.0:0              LISTENING       680
  TCP    192.168.137.131:139    0.0.0.0:0              LISTENING       4
  TCP    192.168.137.131:57918  192.168.137.1:139      TIME_WAIT       0
  TCP    [::]:135               [::]:0                 LISTENING       916
  TCP    [::]:445               [::]:0                 LISTENING       4
  TCP    [::]:5357              [::]:0                 LISTENING       4
  TCP    [::]:49664             [::]:0                 LISTENING       704
  TCP    [::]:49665             [::]:0                 LISTENING       540
  TCP    [::]:49666             [::]:0                 LISTENING       552
  TCP    [::]:49667             [::]:0                 LISTENING       460
  TCP    [::]:49668             [::]:0                 LISTENING       1772
  TCP    [::]:49669             [::]:0                 LISTENING       680
  UDP    0.0.0.0:3702           *:*                                    1688
  UDP    0.0.0.0:3702           *:*                                    1688
  UDP    0.0.0.0:5050           *:*                                    1120
  UDP    0.0.0.0:5353           *:*                                    3788
  UDP    0.0.0.0:5353           *:*                                    1244
  UDP    0.0.0.0:5353           *:*                                    3788
  UDP    0.0.0.0:5355           *:*                                    1244
  UDP    0.0.0.0:53269          *:*                                    1688
  UDP    127.0.0.1:1900         *:*                                    1688
  UDP    127.0.0.1:50120        *:*                                    1688
  UDP    127.0.0.1:53271        *:*                                    460
  UDP    192.168.137.131:137    *:*                                    4
  UDP    192.168.137.131:138    *:*                                    4
  UDP    192.168.137.131:1900   *:*                                    1688
  UDP    192.168.137.131:50119  *:*                                    1688
  UDP    [::]:3702              *:*                                    1688
  UDP    [::]:3702              *:*                                    1688
  UDP    [::]:5353              *:*                                    1244
  UDP    [::]:5353              *:*                                    3788
  UDP    [::]:5355              *:*                                    1244
  UDP    [::]:53270             *:*                                    1688
  UDP    [::1]:1900             *:*                                    1688
  UDP    [::1]:50118            *:*                                    1688
  UDP    [fe80::6067:1152:ba6c:26c7%6]:1900  *:*                                    1688
  UDP    [fe80::6067:1152:ba6c:26c7%6]:50117  *:*                                    1688
```




id = 28
```bash

Aliases for \\DESKTOP-QV3NQLM

-------------------------------------------------------------------------------
*Access Control Assistance Operators
*Administrators
*Backup Operators
*Cryptographic Operators
*Device Owners
*Distributed COM Users
*Event Log Readers
*Guests
*Hyper-V Administrators
*IIS_IUSRS
*Network Configuration Operators
*Performance Log Users
*Performance Monitor Users
*Power Users
*Remote Desktop Users
*Remote Management Users
*Replicator
*System Managed Accounts Group
*Users
The command completed successfully.

```



id = 29
```bash
 SESSIONNAME       USERNAME                 ID  STATE   TYPE        DEVICE 
 services                                    0  Disc                        
>console           Larry Stevens             2  Active                      
```


id = 30-34
```bash
'WMI' is not recognized as an internal or external command,
operable program or batch file.
```





