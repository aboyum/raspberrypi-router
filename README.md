# raspberrypi Cisco-like router
How to make a Cisco like Linux router

All regular linux distributions may install FRRouting effektivly making the computer a advanced router. Many professional "white box routers" such as nvidia's hi-end routers and switches supports FRR, so does Ubuntu, pfsense, Raspberry Pi and many others.

Raspbery Pi 3 contains of 2 networking interfaces. One 2.4 GHz wifi (wlan0) and one 1gbit ethernet interfacce that uses USB2 (eth0) with teoretical 480mbit speed.
In this example wifi0 will be the outside port, with dhcp client. While eth0 will be "LAN side" with a dhcp server.

First enable snap
```
sudo apt update
sudo apt install snapd
sudo reboot
sudo snap install core
```
Install frr
```
sudo snap install frr
```

Become superuser: `sudo su`
Go to `/etc/frr/`
The services is enabled by editig the `daemons` file within /etc/frr/
For instance: To enable ospf go to "ospfd=no" and change the text to "ospfd=yes"

You now have FRR installed and enabled. To configure it by a ssh like "vty-line" like you tyically do on a cisco router: 
Log into the command line by typing `vtysh`
type `exit` to return back to the rasperry os.

Now you may use static ip at your interfaces inside frr like this (not recommended):
```
vtysh
conf term
interface wifi0
ip address 192.168.1.2/24
no shut
exit
write mem
```

However it may be better to 
manually adjust the ip directly in the operating system.

This is configuration static ipdirecly in linux (not in frr):
```
sudo nano /etc/dhcpcd.conf
interface eth0
static ip_address=192.168.1.2
static routers=192.168.1.1
static domain_name_servers=192.168.1.1 8.8.8.8
```
Save and exit Nano (CTRL+W, CTRL+X).
Now, you can restart your Raspberry Pi to apply the changes:
```
sudo reboot
```
Then you may configure more advanced things like configure OSPF or BGP within FRR.
Remember, that for the rest of your network to know about the eth0 network the router need to tell about it. This can be done by a ospf neighbourship (must be configured at both ends)
```
vtysh
conf term
int eth0
ip ospf area 0

router ospf
passive-interface default
no passive-interface eth0

show ip ospf nei
show ip ospf int eth0
```

## DHCP Server
To make a dhcp server you may install dnsmasq
```
sudo apt install dnsmasq
```
Open the DNSMasq configuration file with Nano:
```
sudo nano /etc/dnsmasq.conf
```
Almost everything is commented in this file, and it’s a pretty long file, so the easiest way is to copy and paste these lines at the end (CTRL+_ and CTRL+V):
```
interface=eth0
bind-dynamic
domain-needed
bogus-priv
dhcp-range=192.168.1.100,192.168.1.200,255.255.255.0,12h
```
## OpenVPN client
You may make a OpenVPN connection to encrypt you traffic. You may have a openvpn server at your main location and a Raspberry Pi at a remote location.
Here is how to make a OpenVPN client

```
sudo apt install openvpn
```
make your openvpn file from your openvpnserver

insert the text from the server inside a file in:
```
/etc/openvpn/client
sudo nano /etc/openvpn/client/newfile.conf
```
(the recomended filextension is .conf, not .ovpn)

autostart openvpn if you please
```
sudo nano /etc/default/openvpn
```
and uncomment, or remove, the “#” in front of `AUTOSTART="all"`

The most common OpenVPN (at present 2021) is version 2.4. Many OpenVPN server such as pfsense has got to version 2.5. You may have to mark "support for 2.4" in your server.

A sucessful OpenVPN connecteion adds another interface `tun0`, you may add that to your router configuration as well.
