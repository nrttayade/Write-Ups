## Kioptrix Level 1

So, I am newbie in the field of pentesting and this is going to my first write up. This is my first attempt on Kioptrix and it can be said rather on any pentesting challenge. So, let’s begin.

#### What is Kioptrix?
This Kioptrix VM Image are easy challenges. The object of the game is to acquire root access via any means possible (except actually hacking the VM server or player). The purpose of these games are to learn the basic tools and techniques in vulnerability assessment and exploitation. There are more ways then one to successfully complete the challenges.
Source: https://www.vulnhub.com/entry/kioptrix-level-1-1,22/

### Setup
So, I've set up the Kioptrix as well as Kali Linux on my Virtual Box. Also, both of them are configured for a bridged connection so that they can directly get their own IP Addresses.

### Reconnaissance
At the initial stage we don’t event know the IP address of Kioptrix. So, for that we can run a simple ```netdiscover``` command as given below:

```
root@kali:~# netdiscover -r 192.168.0.0/24

 Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                               
                                                                                                                                                                                                             
 16 Captured ARP Req/Rep packets, from 5 hosts.   Total size: 960                                                                                                                                            
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.1     0c:d2:xx:xx:xx:46      5     300  Router                                                                                                                      
 192.168.0.102   e8:d0:xx:xx:xx:87      3     180  Laptop                                                                                                                             
 192.168.0.100   44:07:xx:xx:xx:68      4     240  Speaker                                                                                                                                              
 192.168.0.101   04:c8:xx:xx:xx:98      2     120  Mobile                                                                                                                            
 192.168.0.108   08:00:xx:xx:xx:59      2     120  PCS Systemtechnik GmbH   
```

From the output above, I knew all the devices except for the one having IP address: 192.168.0.108. So, most probably our Kioptrix would be associated to this IP address only. We run another scan on the same IP, sp as to make sure this IP belongs to Kioptrix. And for that we use the ```nmap``` command:

```
root@kali:~# nmap -T4 -p- 192.168.0.108
Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-11 12:54 IST
Nmap scan report for 192.168.0.108
Host is up (0.000074s latency).
Not shown: 65529 closed ports
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
111/tcp   open  rpcbind
139/tcp   open  netbios-ssn
443/tcp   open  https
32768/tcp open  filenet-tms
MAC Address: 08:00:27:43:90:59 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 4.41 seconds

```

It can be seen that this is a briefing of all the open ports on the particular IP address. The meaning of different switches that we've used are:

-T4: Used for speed, 1 being the slowest and 5 being the fastest
-p-: Scan all the ports

From the output above, we can see several ports are open and it is interesting to see ports like 22, 80, 139 and 443 are open.  These can be vulnerable and we might get our access from one of these ports only. We can run another deep scan to get some more details for the same:

```
root@kali:~# nmap -A -T4 -p22,80,111,139,443,32768 192.168.0.108
Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-11 12:56 IST
Nmap scan report for 192.168.0.108
Host is up (0.00080s latency).

PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 2.9p2 (protocol 1.99)
| ssh-hostkey: 
|   1024 b8:74:6c:db:fd:8b:e6:66:e9:2a:2b:df:5e:6f:64:86 (RSA1)
|   1024 8f:8e:5b:81:ed:21:ab:c1:80:e1:57:a3:3c:85:c4:71 (DSA)
|_  1024 ed:4e:a9:4a:06:14:ff:15:14:ce:da:3a:80:db:e2:81 (RSA)
|_sshv1: Server supports SSHv1
80/tcp    open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
|_http-title: Test Page for the Apache Web Server on Red Hat Linux
111/tcp   open  rpcbind     2 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp   open  ssl/https   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
|_http-server-header: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
|_http-title: 400 Bad Request
|_ssl-date: 2019-11-11T17:57:35+00:00; +10h29m58s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC4_64_WITH_MD5
|_    SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
32768/tcp open  status      1 (RPC #100024)
MAC Address: 08:00:27:43:90:59 (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 2.4.X
OS CPE: cpe:/o:linux:linux_kernel:2.4
OS details: Linux 2.4.9 - 2.4.18 (likely embedded)
Network Distance: 1 hop

Host script results:
|_clock-skew: 10h29m57s
|_nbstat: NetBIOS name: KIOPTRIX, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_smb2-time: Protocol negotiation failed (SMB2)

TRACEROUTE
HOP RTT     ADDRESS
1   0.80 ms 192.168.0.108

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 127.28 seconds
```

There is one more additional switch that is used here and that is the -A which is used to get details about OS, host and other thing.

From the output we can see the version of various services that are running on Kioptrix. Quickly comparing them we the current versions we figured out the following:

* Outdated mod_ssl 2.8.4: Current minimum is 2.8.1
* Outdated OpenSSL 0.9.6b: Current minimum is 1.1.1
* Outdated Apache 1.3.20: Current minimum is 2.8.37
* Samba is running on port 139, we will find the version it is running on using metasploit

