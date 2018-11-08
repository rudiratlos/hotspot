# hotspot

shell script for setup and management for hotspot (hostapd) functions on rpi platform

functions:

- try
- start
- stop 
- status 
- setup [notemplate]
- setchan [channel] 
- modpar \<dnsmasq|hostapd\> \<name\> \<value\>
- wlan [start|stop]

actions will be logged to /tmp/hotspot and syslog 

## installation

rpi login as root required

~~~bash
root:# cd /usr/local/sbin
root:# wget https://raw.githubusercontent.com/rudiratlos/hotspot/master/hotspot
root:# chmod +x hotspot
root:# apt-get update
root:# apt-get upgrade
~~~

## setup

will install all required packages (hostapd and dnsmasq),\
setting parameters and create config files:

- activate net.ipv4.ip_forward=1 in file /etc/sysctl.conf 
- /etc/rc.local_template
- /etc/dhcpcd.conf_template
- /etc/dnsmasq.conf_template
- /etc/default/hostapd_template
- /etc/hostapd/hostapd.conf_template

~~~bash
hotspot setup
~~~

### setup notemplate

create config files without ***_template***.\
***!! This will overwrite existing files !!***

next commands will create all config files and adjusts parameter to your environment.

~~~bash
hotspot setup notemplate
hotspot modpar hostapd ssid myHotspotID 
hotspot modpar hostapd wpa_passphrase myHotspotPassword
hotspot modpar hostapd country DE
~~~

## start

start all hotspot associated functions:

- terminate connection on wlan0
- create device ap0 and assign IP addr
- start dnsmasq
- start hostapd 

~~~bash
hotspot start
~~~

## try

start hotspot if wlan is not connected, or wlan0 and eth0 IP addrs are on same subnet.

~~~bash
hotspot try
~~~

## stop

stop hotspot functions:

- stop hostapd
- stop dnsmasq
- restart wlan

~~~bash
hotspot stop
~~~

## modpar

change parameter value in config file

format:
hotspot modpar \<dnsmasq|hostapd\> \<name\> \<value\>

~~~
file selector:
dnsmasq 		/etc/dnsmasq.conf
hostapd 		/etc/hostapd/hostapd.conf

name			parameter name
value			parameter value
~~~

examples:
~~~bash
hotspot modpar hostapd ssid myHotspotID     # set parameter ssid=myHotspotID
hotspot modpar hostapd country DE           # set parameter country_code=DE
~~~

### special hostapd parameter

#### autostart

During boot process /etc/rc.local will look for file content ***#autostart=1*** and will execute **hotspot try** command.

~~~bash
hotspot modpar hostapd autostart 1          # enable  autostart
hotspot modpar hostapd autostart 0          # disable autostart
~~~

#### useiptables

hotspot script will look for file content ***#useiptables=1*** or ***#useiptables=0*** and will execute **iptables** commands for activation and deactivation.

~~~bash
hotspot modpar hostapd useiptables 1        # executing iptable commands
hotspot modpar hostapd useiptables 0        # no iptable commands
~~~
