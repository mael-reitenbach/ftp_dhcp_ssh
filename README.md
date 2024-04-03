# DHCP, DNS, FTP et SSH

---

## Summary: 
1. ### Virtual environments: 
    Install two Debian 12.5.0 virtual environments, no graphical interface, in a virtual network. Implies immediate system check for updates/update after install.
2. ### DHCP Config:
    Configure a DHCP service on the first machine.
3. ### FTP and SSH: 
    Configure FTP and SSH services on the second machine, then use SSH for SFTP connections.
4. ### DNS Config: 
    Configure a DNS service on the first machine.
5. ### Testing: 
    Conduct a few connection tests.
6. ### Basic security features: 
    Configure a few security features.
7. ### Further security testing:
    Conduct security tests, additionnal security and overall configuration improvement.

Virtual environments
---
---

We start with the configuration of two Debian 12.5.0 virtual environments.    
We use QEMU/KVM, for its reliability, security features, and rapidity.

**Specifications:**
* Hard disk: 15G
* Memory: 2048M
* Network adapter: NAT
* CPU: 1 core

**First machine:**
* Hostname: ns
* Main user: -------

**Second machine:**
* Hostname: server2
* Main user: ------

**Virtual Network Settings**
* Name: network
* IP Address Class: B
* Network IP Address: 172.16.69.0/24
* Gateway: 172.16.69.2
* Transfer Type: NAT

DHCP Configuration
---
---
### On the machine we will use as a DHCP:
We install the package isc-dhcp-server.  
We then check the network cards in the /etc/network/interfaces file.
Here we change the cards IP address and put it to static. We set up a gateway in 172.16.69.2, 
and Google's DNS: 8.8.8.8.
We add those lines at the end of the file, deleting the old interface's configuration. 
Please replace "enp1s0" with your own interface's name.
```shell
auto enp1s0
iface enp1s0 inet static
address 172.16.69.10
netmask 255.255.255.0
gateway 172.16.69.1
dns-nameservers 8.8.8.8
```
We then restart the interface with:
`ifdown enp1s0`
`ifup enp1s0`
In /etc/default/isc-dhcp-server, we specify our interface's name in `INTERFACESv4=""`   

We then configure the dhcp in /etc/dhcp/dhcpd.conf:
We add those lines at the very end of the file:
```shell
subnet 172.16.69.0 netmask 255.255.255.0 {
  range 172.16.69.11 172.16.69.50;
  option domain-name-servers 8.8.8.8;
  option routers 172.16.69.1;
  option broadcast-address 172.16.69.254;
  default-lease-time 600;
  max-lease-time 7200;
}
```
Here we specify the DHCP's options on this subnet, we exclude from the range our own address and the gateway address.
Please note that the gateway address is by default the network address .1, so here it is 172.16.69.1.

After DHCP configuration, we restart the service using  
`sudo systemctl restart networking` first, to restart networking interfaces   
`sudo systemctl restart isc-dhcp-server` we then use 

Now we check the configuration on the other machine:
```shell
auto lo
iface lo inet loopback

allow-hotplug enp1s0
iface enp1s0 inet dhcp
```
To make a new DHCP request, we do the commands:  
`ifdown enp1s0`  
This downs the interface.  
`ifup enp1s0`  
This starts the interface, which will broadcast a DHCP request, and will then get back an IP address from ns.
###### We need to insert here an image/text of the result of the command

We execute the command  
`ip a` which shows us:
###### We need to insert here an image/text of the result of the command

We can see here that the DHCP gave the address 172.16.69.15/24 to our second machine. Meaning everything works fine.

FTP and SSH Configuration
---
---

#### SSH

On the second machine ("server2"), we start by installing the openssh-server package.  
`sudo apt install openssh-server`
We open the configuration file, deactivate root login, and change the default port to 25565 (for example).  
`sudo vi /etc/ssh/sshd_conf`  
_Please note that those operations are not mandatory but recommended as the first steps for basic ssh security._
To connect in ssh to the virtual machine, we use, on our host: ssh -p 25565 ------@172.16.69.15

#### FTP

We first install the proftpd package.  
`sudo apt install openssh-server`  
To configure it, we open the configuration file:  
`sudo vi /etc/proftpd/proftpd.conf`  
Here we uncomment `DefaultRoot ~` to lock the users into their directories, and we change `Port 21 ` to `Port 21999` aswell as `MaxInstances 1` to allow for only one session at a time.  
We don't want people to connect to real users on the machine so we create a virtual user, using:  
`ftpasswd --passwd --file /etc/proftpd/ftpd.passwd --name=************ --change-password`
We then go into the proftpd configuration file and add thoses lines after `DefaultRoot ~`  
```
RequireValidShell off
AuthUserFile /etc/proftpd/ftpd.passwd
AuthOrder mod_auth_file.c
```

It specifies to proftpd that users won't be able to connect with classic shell users in `RequireValidShell off`, the `AuthUserFile /etc/proftpd/ftpd.passwd` indicates to proftpd to look for the virtual users in the ftpd.passwd file.

#### SFTP

Using SFTP is more simple than FTP, as we already have the openssh package installed.
For security purposes, we start by adding a group sftpusers on our server.
We then create the user "------------" which belongs to this group. We redirect its shell to /bin/false with: 
`sudo useradd -m -g sftpusers -s /bin/false ------------`
We change its password with `sudo passwd ------------`
We then go into the sshd_config file, and add/uncomment those lines at the end of the file:
```text
Subsystem sftp internal-sftp

Match Group sftpusers
	X11Forwarding no
	AllowTcpForwarding no
	ChrootDirectory /sftp/%u
	ForceCommand internal-sftp
```
These specify that the sftp group will be sftpusers, with their files being in the directory /sftp/%u (username).
We then create the folder: `sudo mkdir -p /sftp/laplateforme/files`  
And give the right permissions:
```text
sudo chown root:sftpusers /sftp/laplateforme
sudo chmod 755 /sftp/laplateforme
sudo chown laplateforme:sftpusers /sftp/laplateforme/files/
```
Finally, we restart the ssh service: `sudo systemctl restart ssh`
We can now connect to the server through sftp using a client like FileZilla.

Testing
---
---

We can conduct a few tests to see if everything works fine:  
We try root connection in ssh to the server: it should not be allowed.  
We try root/any real user connection through ftp: it should not be allowed.  
We make sure virtual user connection works through ftp.  
We try to connect through sftp with real user without sftpusers group permissions: it should not be allowed.  
We make sure real user with sftpusers group permissions connection works through sftp.

Basic security features
---
---

Those are the ones we already implemented: segmentation of real/virtual users for FTP/SFTP, permissions and groups, specific directories for each user through FTP/SFTP.  
Changing the default ports to avoid bots mapping our server, disable root login with either protocols.

Further security testing
---
---

We can try nmapping our machines to see what information we can gather about them.   
We can check iptables configuration and filter ports.
