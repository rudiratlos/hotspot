# hotspot

shell script for setup and management of hotspot (hostapd) functions on rpi platform

functions:

- try
- start
- stop
- restart
- retry
- status
- setup [notemplate]
- setchan [channel]
- modpar \<dnsmasq|hostapd\> \<name\> \<value\>
- wlan [start|stop]

actions will be logged to /tmp/hotspot and syslog\
pls. see examples in troubleshooting section

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

create config files without filename part ***_template***.\
***!! This will overwrite existing files !!***

~~~bash
hotspot setup notemplate

hotspot try
~~~

above command sequence will create hotspot with following default parameter:

ssid:    \<HOSTNAME\>wlan-\<MAC3ByteAdr\> (e.g. RPIwlan-abcdef)\
pwd:     hallohallo\
country: DE\


next commands will create all config files and adjusts parameter to your environment.

~~~bash
hotspot setup notemplate
hotspot modpar hostapd ssid myHotspotID 
hotspot modpar hostapd wpa_passphrase myHotspotPassword
hotspot modpar hostapd country SE

hotspot try
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

## stop [nowlan]

stop hotspot functions:

- stop hostapd
- stop dnsmasq
- optional: restart wlan

~~~bash
hotspot stop
~~~

## restart

executes following hotspot sequence:

- hotspot stop nowlan
- sleep 20 seconds (settling time)
- hotspot start

~~~bash
hotspot restart
~~~

## retry

executes following hotspot sequence:

- hotspot stop nowlan
- sleep 20 seconds (settling time)
- hotspot try

~~~bash
hotspot retry
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
hotspot modpar hostapd country_code DE      # set parameter country_code=DE
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

## troubleshooting

log entries will be sent to the file /tmp/hotspot.log and syslog utility

following commands will show you hotspot script activity

~~~bash
cat /tmp/hotspot.log
tail -500 /var/log/syslog | grep -a "hotspot:"
cat /var/log/syslog | grep -a "hotspot:"
~~~

these commands will show 5 log entries of involved SW packages caused by hotspot command sequence

~~~bash
tail -500 /var/log/syslog | grep -a -A 5 "hotspot:"
cat /var/log/syslog | grep -a -A 5 "hotspot:"
~~~
