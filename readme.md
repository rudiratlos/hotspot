# hotspot

shell script for setup and management for hotspot (hostapd) functions on rpi platform

functions:

- try
- start
- stop 
- restart
- status 
- setup [notemplate]
- autostart <0|1>
- useiptables <0|1>
- country <XX>
- ssid <ssid>
- pwd <pwd> 
- setchan [channel] 
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

~~~bash
hotspot setup notemplate
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

## set parameter in /etc/hostapd/hostapd.conf

set specific parameters in file ( /etc/hostapd/hostapd.conf ).\

### autostart

During boot process /etc/rc.local will look for file content ***#autostart=1*** and will execute **hotspot try** command.

~~~bash
hotspot autostart 1      	# enable  autostart
hotspot autostart 0      	# disable autostart
~~~

### country

~~~bash
hotspot country DE			# set parameter country_code=DE
~~~

### pwd

~~~bash
hotspot pwd changeme		# set parameter wpa_passphrase=changeme
~~~

### ssid

~~~bash
hotspot ssid myHotSpotSSID	# set parameter ssid=myHotSpotSSID
~~~

### useiptables

hotspot script will look for file content ***#useiptables=1*** or ***#useiptables=0*** and will execute **iptables** commands for activation and deactivation.

~~~bash
hotspot useiptables 1      # executing iptable commands
hotspot useiptables 0      # no iptable commands
~~~
