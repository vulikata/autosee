# Softether

[![N|Solid](https://www.softether.org/@api/deki/files/5/=selogo.jpg)](https://www.softether.org/)


SoftEther is a powerful multi-protocol VPN software and easy to use. He is compatible with all operatins systems (Windows, Linux, Mac, FreeBSD, Solaris...). A very good thing with SoftEther is that it is an open source, so we can use it for any personal or commercial use for free charge without limitation in connections. It is compatible with OpenVPN, L2TP/IPsec, SSTP protocols. SoftEther has also his own protocol VPN that is faster than OpenVPN, L2TP/IPsec and SSTP protocols.
# Softether Auto Installation


### Prerequisites
- Virtual Private Server [DigitalOcean](http://digitalocean.com), [AWS](https://aws.amazon.com), [Google Cloud](https://cloud.google.com/)
- Ubuntu 16.04 x64 or Debian 8.10 x64
- Root privilege
- Putty
- Auto Script


### Big credits to this awesome people
-  [Captain Underpants](https://www.phcorner.net/members/755578/)
- u s n e k x
- [STnetwork](https://github.com/STnetwork)
- [Lincoln Lee](https://github.com/linc01n)


### Instruction
Copy and Paste this script <pre lang="no-highlight">```wget https://raw.githubusercontent.com/iamzildjian/SEscript/master/seautoscriptupdated && chmod +x seautoscriptupdated && ./seautoscriptupdated```</pre>into your terminal


Just fillup the required informations

Enter Server IP:<br /> 
Set Virtual Hub:<br />
Set Hub username:<br />
Set Hub password:<br />
Set SE server password: (your softether server manager password)

Wait for the installation to finish and you're ready to go. 

![](https://i.imgur.com/l7C2Ues.png)

If you wish to use local bridge connection, follow the procedure below as the method above configured as SNAT. 


# Configuration Using Local Bridge

For this step we're not going to use SecureNAT and SecureDHCP. Instead, we are going to use dnsmasq as our DHCP server.


After the installation of the script you should disable the secure NAT inside the virtual NAT and virtual DHCP server as the script enable it automatically.


![](https://i.imgur.com/0Hj9XoK.png)
![](https://i.imgur.com/aIIMRhJ.png)


After disabling secureNAT go to local bridge settings. 


![](https://i.imgur.com/NZDAXKO.png)


Inside local bridge choose the virtual hub name then check bridge with new tap device. 


![](https://i.imgur.com/T8qIqQ4.png)


Now, for new tap device name you can put any name that you want. But in my case i named it as "soft". 
After that click create local bridge and exit. 


# Installation & Configuration of DNSmasq for a DHCP.


After configuring the local bridge go back to your terminal and run this command.

```ifconfig tap_soft```


You need to install a DHCP server on your server. So we are going to use dnsmasq as our DHCP server.

```apt-get -y install dnsmasq```


Next, edit your /etc/dnsmasq.conf using this command. 

```nano /etc/dnsmasq.conf```

and add these lines at the end.
```
interface=tap_soft
dhcp-range=tap_soft,192.168.7.50,192.168.7.60,12h
dhcp-option=tap_soft,3,192.168.7.1
port=0 
dhcp-option=option:dns-server,208.67.222.222,208.67.220.220
```

![](https://i.imgur.com/sEoQKCt.png)


In your keyboard press ctrl+X then type Y and enter to save.


Next step you need this new set of init script which will config tap interface when Softether start up.
Edit your ```/etc/init.d/vpnserver``` by typing this command.

```nano -w /etc/init.d/vpnserver```


Copy and replace all codes with this.

```#!/bin/sh
### BEGIN INIT INFO
# Provides:          vpnserver
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start daemon at boot time
# Description:       Enable Softether by daemon.
### END INIT INFO
DAEMON=/usr/local/vpnserver/vpnserver
LOCK=/var/lock/subsys/vpnserver
TAP_ADDR=192.168.7.1

test -x $DAEMON || exit 0
case "$1" in
start)
$DAEMON start
touch $LOCK
sleep 1
/sbin/ifconfig tap_soft $TAP_ADDR
;;
stop)
$DAEMON stop
rm $LOCK
;;
restart)
$DAEMON stop
sleep 3
$DAEMON start
sleep 1
/sbin/ifconfig tap_soft $TAP_ADDR
;;
*)
echo "Usage: $0 {start|stop|restart}"
exit 1
esac
exit 0
```
Press Ctrl+X then type Y and enter to save.


Next, add this line ```net.ipv4.ip_forward = 1``` to your ipv4_forwarding.conf directory.


To do that, edit your /etc/sysctl.conf using this command.

```nano -w /etc/sysctl.conf```


Or just copy and paste below codes. It will automatically add the line.

```echo 'net.ipv4.ip_forward = 1' > /etc/sysctl.d/ipv4_forwarding.conf```


![](https://i.imgur.com/RF3SPwu.png)

Now run this command to enable.

```sysctl --system```


Next add a POSTROUTING rule to iptables. 

```iptables -t nat -A POSTROUTING -s 192.168.7.0/24 -j SNAT --to-source [YOUR SERVER IP ADDRESS]```


![](https://i.imgur.com/fuLKeAT.png)

To make the iptables rule survive after reboot you need to install ```iptables-persistent```

```apt-get -y install iptables-persistent```


Choose Yes 2 times to proceed by pressing enter key. 


![](https://i.imgur.com/LJ6iiNx.png)


Then last, run this commands to restart your vpnserver and dhcp server.

```/etc/init.d/vpnserver restart && /etc/init.d/dnsmasq restart```


And you're done.


![](https://i.imgur.com/on1XcuP.png)
