# hotspot

shell script for setup and management of hotspot (hostapd) functions on rpi platform

functions:

- try
- start
- stop
- restart
- enable
- disable
- retry
- status
- setup
- setchan [channel]
- syslog [lines]
- fwd [wipe|install|list]
- ovpn [start|stop|refresh]
- wg [start|stop|config]
- tor [start|stop]
- version
- wlan [start|stop]
- modpar \<dnsmasq|hostapd|self\> \<name\> [value]
- automatic start at boot process by systemd

will use onboard wlan adaptor for hotspot functionality and\
the on board ethernet port or an optional external usb wlan adaptor (e.g. EW-7811Un Realtek RTL8188CUS)\
for internet access.

best wlan channel for hotspot functionality will be determined automatically by least used frequency spectrum.

support for vpns: wireguard and openvpn\
support for tor

create .ovpn config files for free openvpn server taken from [https://www.vpngate.net](https://www.vpngate.net)

actions will be logged in syslog\
pls. see examples in troubleshooting section.

new hotspot version is using firewalld.

for full installation and setup sequence, pls. see **installation and setup** section at the bottom of this file

# fwd

is a small script, that allows to have a jump start with firewalld

## installation

rpi login as root required

~~~bash
root:# cd /usr/local/sbin
root:# wget -qN https://raw.githubusercontent.com/rudiratlos/hotspot/master/hotspot
root:# chmod +x hotspot
root:# apt-get update
root:# apt-get upgrade
~~~

## setup

will install all required packages (e.g. iw tor openvpn wireguard hostapd dnsmasq),\
setting parameters and create config files:

- /etc/sysctl.conf (activate line net.ipv4.ip_forward=1) 
- /etc/rc.local
- /etc/issues
- /etc/dhcpcd.conf
- /etc/dnsmasq.conf
- /etc/default/hostapd
- /etc/hostapd/hostapd.conf
- /etc/tor/torrc
- /etc/wireguard/
- /etc/openvpn/

Existing files will be backed up with a date extension (YYYYMMDDhhmmss). 

~~~bash
hotspot setup
hotspot try
~~~

above command sequence will create a hotspot with following default parameter:

ssid:    \<HOSTNAME\>wlan-\<MAC3ByteAdr\> (e.g. RPIwlan-abcdef)\
pwd:     hallohallo\
country: DE


next commands will create all config files and adjusts parameter to your environment.

~~~bash
hotspot setup
hotspot modpar hostapd ssid myHotspotID 
hotspot modpar hostapd wpa_passphrase myHotspotPassword
hotspot modpar hostapd country_code SE
hotspot modpar crda REGDOMAIN SE

hotspot try
~~~

### tor opvn install disable

before executing **hotspot setup** command, \
you can disable the installation of tor and/or ovpn package by modifying the ***aptaddinstlist*** variable.

~~~bash
hotspot modpar self aptaddinstlist "tor"                    # install tor only
hotspot modpar self aptaddinstlist "openvpn"                # install openvpn only
hotspot modpar self aptaddinstlist "wireguard"              # install wireguard only 
hotspot modpar self aptaddinstlist "tor openvpn wireguard"  # install all three (default)
hotspot modpar self aptaddinstlist ""                       # do not install vpns
~~~

## enable

enable hotspot service

hotspot will be started by systemd service \
control file: /lib/systemd/system/hotspot.service \
the service will be started at boot stage where \
network interfaces are available. \
If autostart variable is set to "yes", hotspot will be tried to start.

Older versions (V0.960), hotspot was started by rc.local

~~~bash
hotspot enable
~~~

## disable

disable hotspot service

~~~bash
hotspot disable
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

will start hotspot if following condition is met:

- wlan0 or eth0 not connected
- wlan0 and eth0 IP addresses are on same IP subnet (wlan0 connection will be stopped)

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
- sleep some seconds (settling time)
- hotspot start

~~~bash
hotspot restart
~~~

## retry

executes following hotspot sequence:

- hotspot stop nowlan
- sleep some seconds (settling time)
- hotspot try

~~~bash
hotspot retry
~~~

## modpar

change parameter value in config file

format:
hotspot modpar \<dnsmasq|hostapd|self\> \<name\> [value]

~~~
file selector:
dnsmasq                 /etc/dnsmasq.conf
hostapd                 /etc/hostapd/hostapd.conf
self                    /usr/local/sbin/hotspot

name			parameter name
value			parameter value
~~~

examples:
~~~bash
hotspot modpar hostapd ssid myHotspotID      # set parameter ssid=myHotspotID
hotspot modpar hostapd country_code DE       # set parameter country_code=DE
~~~

### special hostapd parameter

#### autostart

During boot process systemd service hotspot.service will look for \
file content ***autostart="yes"*** in /usr/local/sbin/hotspot and will execute **hotspot try** command.\

V0.960: This is not needed anymore, use ***hotspot enable/disable*** command.

~~~bash
hotspot modpar self autostart yes            # enable  autostart (default)
hotspot modpar self autostart no             # disable autostart
~~~

#### ovpnstart

start openvpn automatically

~~~bash
hotspot modpar self ovpnstart yes           # enable  ovpnstart
hotspot modpar self ovpnstart no            # disable ovpnstart (default)
~~~

adjust specific openvpn parameter

~~~bash
hotspot modpar self ovpn_dev tun3           # change ovpn device for firewalld
~~~

to work correctly, ovpn_dev has to be the same, that is defined in .ovpn config file (parameter dev). 

#### torstart

start tor service automatically

~~~bash
hotspot modpar self torstart yes            # enable  torstart
hotspot modpar self torstart no             # disable torstart (default)
~~~

#### wgstart

start wireguard service automatically

~~~bash
hotspot modpar self wgstart yes            # enable  wgstart
hotspot modpar self wgstart no             # disable wgstart (default)
~~~

## openvpn (user specific)

copy myconfig.ovpn file to /etc/openvpn/myconfig.conf

~~~bash
cp myconfig.ovpn /etc/openvpn/myconfig.conf 
hotspot modpar self ovpncfg myconfig
hotspot modpar self ovpnrefreshbeforestart no
hotspot modpar self ovpnstart yes
~~~

pls. assure, that the your config file (/etc/openvpn/myconfig.conf) has the extension .conf\
and set parameter 'ovpncfg myconfig'\
the script will start a service **openvpn@myconfig**

## openvpn (vpngate)

start, stop openvpn or refresh .ovpn files from vpngate.net **experimental**\
refresh will download the CSV list of free openvpn server and will create .ovpn files.\
server from these countries will be used, defined by **ovpnsel** parameter: **AT CH DE ES FR GB JP KR SC TW US**
out of these, the server with the highest score is defined in /etc/openvpn/client/vpngate_bestscore.ovpn\
and will be copied to **/etc/openvpn/hotspot.conf** and used as default openvpn config file.\
the script will start a service **openvpn@hotspot** if parameter **ovpncfg** is set to **hotspot**

pls. see ***ovpnstart*** parameter for automatic starting openvpn and modifying parameter

~~~bash
hotspot modpar self ovpncfg hotspot         # set default config name
hotspot ovpn start                          # start openvpn service
hotspot ovpn stop                           # stop  openvpn service (default)
hotpsot ovpn refresh                        # recreate .ovpn config files
~~~

## tor

start or stop tor service **experimental**\
pls. see ***torstart*** parameter for automatic starting tor service.

~~~bash
hotspot tor start                           # start tor service
hotspot tor stop                            # stop  tor service
~~~

## wireguard (experimental)

setup example for wg client, which should connect to remote wireguard server (wg.example.com:51820)

~~~bash
                                                  # set default values
hotspot modpar self wg_clientprivkey "uME..EGE="  # [Interface] PrivateKey
hotspot modpar self wg_srvpubkey     "URS..PQY="  # [Peer] PublicKey 
hotspot modpar self wg_presharedkey  "6Oo..qOs="  # [Peer] PresharedKey or leave blank
hotspot modpar self wg_endpoint "wg.example.com:51820"  # [Peer] Endpoint
hotspot modpar self wgstart yes  # will start wireguard tunnel at startup

hotspot wg config client   # create interactively wg config file /etc/wireguard/wg0.conf

hotspot wg start           # start wireguard service
hotspot wg stop            # stop  wireguard service
~~~

## syslog [lines]

show hotspot related syslog entries

~~~bash
hotspot syslog
hotspot syslog 5
~~~

## version

show hotspot script version

~~~bash
hotspot version
~~~

## installation and setup

rpi login as root required

~~~bash
root:# cd /usr/local/sbin

root:# cp hotspot hotspot.old               # if you want to keep it
root:# rm hotspot                           # just remove old hotspot script
root:# wget -qN https://raw.githubusercontent.com/rudiratlos/hotspot/master/hotspot
root:# chmod +x hotspot

root:# cp fwd fwd.old                       # if you want to keep it
root:# wget -qN https://raw.githubusercontent.com/rudiratlos/hotspot/master/fwd
root:# chmod +x fwd

root:# apt-get update
root:# apt-get upgrade                      # optional

root:# hotspot setup

root:# hotspot modpar hostapd ssid myHotspotID 
root:# hotspot modpar hostapd wpa_passphrase myHotspotPassword
root:# hotspot modpar hostapd country_code SE
root:# hotspot modpar crda REGDOMAIN SE
root:# hotspot modpar self autostart yes    # optional autostart enable

root:# reboot                               # if autostart enable or use hotspot try
~~~

## troubleshooting

syslog will be used as log utility

~~~bash
hotspot syslog
tail -500 /var/log/syslog | grep -a "hotspot"
cat /var/log/syslog | grep -a "hotspot"
~~~

these commands will show 5 log entries of involved SW packages caused by hotspot command sequence

~~~bash
hotspot syslog 5
tail -500 /var/log/syslog | grep -a -A 5 "hotspot"
cat /var/log/syslog | grep -a -A 5 "hotspot"
~~~
