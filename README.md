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
* Main user: meijden


**Second machine:**
* Hostname: server2
* Main user: pierre


**Virtual Network Settings**
* Name: network
* IP Address Class: B
* Network IP Address: 172.16.69.0
* Gateway: 172.16.69.2
* Transfer Type: NAT


DHCP Configuration
---
---
### On the machine we will use as a DHCP:
We install the package isc-dhcp-server. 
We then check the network cards in the /etc/network/interfaces file.
We change the used card with a static IP address.


