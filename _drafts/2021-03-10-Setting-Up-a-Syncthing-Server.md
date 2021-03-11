---
title: Setting Up a Syncthing Server
permalink: /technology/syncthing
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/isCXA9KCfvA" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


I had the opportunity to hold my own presentation and workshop for Cal Poly SWIFT last week. My presentation covered what servers were,  introduced Syncthing, file synchronization , and  and the workshop covered how to host a server capable of syncing files across different devices using Syncthing. 

## SSH
```shell=
ssh-keygen -t ed25519
```
This command creates an SSH keypair located in ```/home/<username>/.ssh```

```shell=
ls -la ~/
```
The dot in front of the folder means it is a hidden folder. If we want to see it, we use ```ls -la```

The ```~``` signifies the ```/home/<username>``` directory

```shell=
ssh-copy-id -i ~/.ssh/<PUBLIC key> <user>@<IP address>
```
This command copies our PUBLIC key to another computer

```shell=
ssh -i </path/to/public key> <user>@<IP address>
```
This command allows us to SSH into another computer

### sshd_config
File location: ```/etc/ssh/sshd_config```

We're looking for these 4 lines in the file:
```shell=
#PermitRootLogin prohibit-password
#MaxAuthTries 6
...
#PubkeyAuthentication yes
...
#PasswordAuthentication yes
```

The # are the default settings, so we're going change these settings to:
- ```PermitRootLogin no```
- ```MaxAuthTries 3```
- ```PasswordAuthentication no```

Remove the # when changing these values and save the file after editting

Then run ```sudo systemctl restart sshd``` to restart the SSH daemon for the changes to take effect

## Installing Syncthing
```shell=
sudo apt update
sudo apt install syncthing
```
```sudo apt update``` refreshes our repository aka our "playstore" for the latest apps

```sudo apt install syncthing``` installs the package "syncthing" from the playstore

## Adding a User to Run the Syncthing Service
We want to add a seperate user to run this service because of the defense in depth approach. IF Syncthing were to be compromised, we don't want the threat to also gain access to a super user/root account.
```shell=
sudo adduser syncthing
```
There is going to be a prompt asking for this new user's password. You can skip the user information section.

## Starting Syncthing
```shell=
sudo systemctl enable syncthing@syncthing.service
sudo systemctl start syncthing@syncthing.service
```

```sudo systemctl enable syncthing@syncthing.service``` will start Syncthing on bootup

```sudo systemctl status syncthing@syncthing.service``` will show us it's current status

```sudo systemctl start syncthing@syncthing.service``` will start Syncthing

## Connecting Other Devices to the Server
Since we are running Syncthing on a server, our GUI that would've been normally accessible over a web browser won't work

There are 2 ways to make our server work:
1. Allow Syncthing to listen on 0.0.0.0
2. SSH tunneling

SSH tunneling is the preferred method, because our web GUI isn't going to be out in the open. We already enabled public key authentication, so only devices that *can* SSH into the server can access the web GUI for configuration.

On your local machine:
```shell=
ssh -L <listening port>:<remote IP address>:<remote port> <remote username>@<remote IP>
```

```-L```: local port forwarding

```<listening port>```: the port you will be listening on, on your HOST machine

```<remote IP address>```: IP of what you want to remote into

```<remote port>```: remote port that you want to tunnel into the listening port 


## ufw
```shell=
sudo ufw enable
sudo ufw allow ssh
```

The last thing we want to do is enable our firewall. This allows our server to be secure from unwanted connections.

```sudo apt install ufw``` if it isn't installed

```sudo ufw enable``` activates the firewall

```sudo ufw status``` shows us our firewall rules

```sudo ufw status vebose``` gives us more details

Activating the firewall for the first time turns on 2 default rules:
- Deny ALL incoming traffic
- Allow ALL outgoing traffic

These two rules are the baseline of ufw, and it allows finer control of what type of traffic you want the server to accept

We are going to add ```sudo ufw allow ssh```, since that is the only incoming traffic our server should be expecting

### Useful ufw commands
```sudo ufw deny <port>``` to deny a port

There are a 2 ways to delete firewall rules:

```sudo ufw status numbered```, then ```sudo ufw delete <#>``` to delete a specific rule

```sudo ufw delete allow 22```

