# hotspot V0.1

shell script for setup and management for hotspot (hostapd) functions on rpi platform

functions:

- try
- start
- stop 
- restart
- status 
- setup [notemplate]
- autostart [enable|disable] 
- iptables [add|del] 
- setchan [channel] 
- wlan [start|stop]

actions will be logged to /tmp/hotspot and syslog 

## installation

rpi login as root required

~~~bash
root:# cd /usr/local/sbin
root:# wget https://raw.githubusercontent.com/rudiratlos/hotspot/master/hotspot
root:# chmod +x hotspot
~~~

## setup

will install all required packages (hostapd and dnsmasq),\
and create config files:

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

## restart

stop hotspot and start hotspot

~~~bash
hotspot restart
~~~

## autostart

just create or delete a flag file ( /etc/default/hostapd_autostart ).\
During boot process /etc/rc.local will look for existance of this file and will execute ***hotspot try*** command.

