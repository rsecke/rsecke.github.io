---
layout: single
title: "Tools and Techniques (T&T)"
sidebar:
    nav: T&T-navigation
classes: wide
permalink: /T&T
---

T&T is a central knowledge base for any tool or technique that I have used so far. This is also the reference point for any tools or techniques that I use in blog posts.

# Tools

## nmap

`nmap` is a network discovery tool that is used to scan and find assets on a network. This tool provides visibility on what devices are on a network, and what ports are running on those devices. `nmap` is able to get detailed results on what service is running on a specific port, and provides scripts for many other detection use cases. Other features include version detection, operating system detection, network information (DNS name, device type, MAC address), and vulnerability detection.

### Common Flags

```
-sC: runs default (safe) scripts on a port
-sV: service versioning
-p-: scans ALL 65535 ports
-oN: output file (-oA will output the scan results in a .xml, .nmap, and .gmap file)
-sn: ping sweep
```

### Sample Output

#### nmap scan

```lua
# Nmap 7.92 scan initiated Tue Nov 23 15:49:13 2021 as: nmap -Pn -sCV -p- -oN buff 10.10.10.198
Nmap scan report for 10.10.10.198
Host is up (0.074s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE    VERSION
7680/tcp open  pando-pub?
8080/tcp open  http       Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
|_http-title: mrb3n's Bro Hut
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Nov 23 15:53:35 2021 -- 1 IP address (1 host up) scanned in 261.93 seconds
```

#### ping sweep

`nmap` can be used to quickly discover assets on a network. This scan is a pinging (sending packets) the `172.16.103.0/24` subnet to see all the live hosts on this network.

```lua
[tyco㉿4YE: ~]$ nmap -sn 172.16.103.0/24

Starting Nmap 7.80 ( https://nmap.org ) at 2020-11-06 22:47 PST
Nmap scan report for 172.16.103.1
Host is up (0.010s latency).
Nmap scan report for 172.16.103.2
Host is up (0.018s latency).
Nmap scan report for 172.16.103.10
Host is up (0.010s latency).
Nmap scan report for 172.16.103.37
Nmap done: 256 IP addresses (4 hosts up) scanned in 4.44 seconds
```

If there is a hostname, `nmap` will pick that up as well

```lua
[tyco㉿4YE: ~]$ nmap -sn 172.16.103.0/24

Starting Nmap 7.80 ( https://nmap.org ) at 2020-11-06 22:47 PST
Nmap scan report for <hostname> (172.16.103.1)
Host is up (0.010s latency).

Nmap done: 256 IP addresses (1 host up) scanned in 4.44 seconds
```

#### script scan

A script scan will scan a target for a specific purpose. All the scripts can be found in `/usr/share/nmap/scripts/` on Kali, and the Nmap Scripting Engine (NSE) [documentation](https://nmap.org/nsedoc/) can be used to provide more information on a script's use case.

```lua
[tyco㉿4YE: ~]$ nmap --script ftp-vsftpd-backdoor -p 21 <host>

PORT   STATE SERVICE
21/tcp open  ftp
| ftp-vsftpd-backdoor:
|   VULNERABLE:
|   vsFTPd version 2.3.4 backdoor
|     State: VULNERABLE (Exploitable)
|     IDs:  CVE:CVE-2011-2523  BID:48539
|     Description:
|       vsFTPd version 2.3.4 backdoor, this was reported on 2011-07-04.
|     Disclosure date: 2011-07-03
|     Exploit results:
|       The backdoor was already triggered
|       Shell command: id
|       Results: uid=0(root) gid=0(root) groups=0(root)
|     References:
|       https://www.securityfocus.com/bid/48539
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-2523
|       http://scarybeastsecurity.blogspot.com/2011/07/alert-vsftpd-download-backdoored.html
|_      https://github.com/rapid7/metasploit-framework/blob/master/modules/exploits/unix/ftp/vsftpd_234_backdoor.rb
```

## gobuster

`gobuster` is a directory brute forcer. This tool is used to find valid directories, subdomains, and virtual hosts on web servers that run more than one website. 

### Common Flags

You supply `gobuster` with the module to use (`dns`, `dir`, `vhost`), a URL, and a wordlist.

```
MODULES
dns: uses the DNS module to scan for subdomains
dir: uses the directory module to scan for directories
vhost: uses the vhost module to scan for other websites running on the webserver

DIR
-u: specify URL
-w: specify wordlist
-k: ignore TLS certificate warning
-x: specify file extensions to search for (e.g. note.txt)
-o: outputs scan results to a file
-t: specifies number of threads to use
-r: follows redirects

DNS
-d: specify domain
-w: specify wordlist
-i: show IPs
-c: show CNAME records (can't be used with -i)

VHOST
-u: specify URL
-w: specify wordlist
```

### Sample Output

#### `dir` scan

```
[tyco㉿4YE: ~]$ gobuster dir -u https://buffered.io -w ~/wordlists/shortlist.txt

===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Mode         : dir
[+] Url/Domain   : https://buffered.io/
[+] Threads      : 10
[+] Wordlist     : /home/oj/wordlists/shortlist.txt
[+] Status codes : 200,204,301,302,307,401,403
[+] User Agent   : gobuster/3.1.0
[+] Timeout      : 10s
===============================================================
2019/06/21 11:49:43 Starting gobuster
===============================================================
/categories (Status: 301)
/contact (Status: 301)
/posts (Status: 301)
/index (Status: 200)
===============================================================
2019/06/21 11:49:44 Finished
===============================================================
```

#### `dns` scan

```
[tyco㉿4YE: ~]$ gobuster dns -d google.com -w ~/wordlists/subdomains.txt

===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Mode         : dns
[+] Url/Domain   : google.com
[+] Threads      : 10
[+] Wordlist     : /home/oj/wordlists/subdomains.txt
===============================================================
2019/06/21 11:54:20 Starting gobuster
===============================================================
Found: chrome.google.com
Found: ns1.google.com
Found: admin.google.com
Found: www.google.com
Found: m.google.com
Found: support.google.com
Found: translate.google.com
Found: cse.google.com
Found: news.google.com
Found: music.google.com
Found: mail.google.com
Found: store.google.com
Found: mobile.google.com
Found: search.google.com
Found: wap.google.com
Found: directory.google.com
Found: local.google.com
Found: blog.google.com
===============================================================
2019/06/21 11:54:20 Finished
===============================================================
```

#### `vhost` scan

```
[tyco㉿4YE: ~]$ gobuster vhost -u https://mysite.com -w common-vhosts.txt

===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:          https://mysite.com
[+] Threads:      10
[+] Wordlist:     common-vhosts.txt
[+] User Agent:   gobuster/3.1.0
[+] Timeout:      10s
===============================================================
2019/06/21 08:36:00 Starting gobuster
===============================================================
Found: www.mysite.com
Found: piwik.mysite.com
Found: mail.mysite.com
===============================================================
2019/06/21 08:36:05 Finished
===============================================================
```

# Techniques

