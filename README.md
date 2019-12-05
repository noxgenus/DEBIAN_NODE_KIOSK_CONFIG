# DEBIAN 9 STRETCH NODEJS KIOSK OS/BROWSER INSTALL+CONFIG 2019

1. Install Debian 9 Stretch from USB and give hostname with correct PC ID<br>
2. Config network if needed<br>
3. Disable apt cd rom in /etc/apt/sources.list<br>
4. Update && Upgrade through internets<br>
5. Add user ‘whoeveryouwant’<br>
6. Apt-get install sudo<br>
7. Add youruser to sudoers<br>
8. Apt-get install -y chromium<br>
9. Goto applications > XFCE Power manager<br>
10. General: power button: shutdown / all others do nothing<br>
11. General: uncheck ‘show notifications’ <br>
12. Display: uncheck ‘handle display power management’ / blank after: never<br>
13. Security: auto lock: never / uncheck lock screen when system sleep<br>

### Edit: 
```
$pico /etc/lightdm/lightdm.conf
```
### Add to bottom:
```
[SeatDefaults]
autologin-user=youruser
user-session=openbox-session
```	

### Then add chromium startup commands to “Applications > Settings > Session and Startup” Tab: Application Autostart.

## TOP MONITOR KIOSK COMMAND (Secondary/right):
```
chromium --no-first-run --disable-translate --noerrordialogs --window-position=1920,0 --window-size=1920,1080 --incognito --disable-session-crashed-bubble --disable-infobars --kiosk http://localhost:8080/top_index.html --no-sandbox --use-fake-ui-for-media-stream
```
<br>

## BOTTOM MONITOR KIOSK COMMAND (Primary/left):
```
chromium "--user-data-dir=${HOME}/.chromium/session${DISPLAY}" --window-position=0,0 --window-size=1920,1080 --no-first-run --incognito --disable-translate --noerrordialogs --disable-session-crashed-bubble --disable-infobars --kiosk --no-sandbox --use-fake-ui-for-media-stream http://localhost:8080/index_bottom.html
```





## MANUAL Xcfe4 monitor config (FORCED):
```
$pico /home/youruser/.config/xfce4/xfconf/xfce-perchannel-xml/displays.xml
```

```
<?xml version="1.0" encoding="UTF-8"?>

<channel name="displays" version="1.0">
  <property name="Default" type="empty">
    <property name="DVI-D-1" type="string" value="1. Samsung 48&quot;">
      <property name="Active" type="bool" value="true"/>
      <property name="Resolution" type="string" value="1920x1080"/>
      <property name="RefreshRate" type="double" value="60.000000"/>
      <property name="Rotation" type="int" value="0"/>
      <property name="Reflection" type="string" value="0"/>
      <property name="Primary" type="bool" value="true"/>
      <property name="Position" type="empty">
        <property name="X" type="int" value="0"/>
        <property name="Y" type="int" value="0"/>
      </property>
    </property>
    <property name="HDMI-2" type="string" value="2. Samsung 48&quot;">
      <property name="Active" type="bool" value="true"/>
      <property name="Resolution" type="string" value="1920x1080"/>
      <property name="RefreshRate" type="double" value="60.000000"/>
      <property name="Rotation" type="int" value="0"/>
      <property name="Reflection" type="string" value="0"/>
      <property name="Primary" type="bool" value="false"/>
      <property name="Position" type="empty">
        <property name="X" type="int" value="1920"/>
        <property name="Y" type="int" value="0"/>
      </property>
    </property>
  </property>
  <property name="Notify" type="bool" value="true"/>
</channel>
```

## REMOVE FUCKING GNOME RING:
```
$apt-get remove gnome-keyring
```

## POWER BUTTON GRACEFUL SHUTDOWN:
```
apt-get -y install acpid
Edit /etc/systemd/logind.conf 
```
and uncomment:
```
HandlePowerKey=poweroff
```


## NODE setup
```
$sudo apt-get install curl
$curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
$nvm install 6
$npm install (from packages)
```

## TTY Support
```
$sudo gpasswd --add ${USER} dialout
$sudo gpasswd --add ${USER} tty
$shutdown -r
```


## Boot node script on startup:
```
$ sudo npm install -g forever
$ sudo npm install -g forever-service
```

## To install NodeJS script as a service:
```
$ cd /home/youruser/nodejs/yournodejsproject/
$ sudo forever-service install yourservice --script yournodescript.js
```

## FOREVER SERVICE LOG FILE: /var/log/vansservice.log

### Install imageMagick for GIF rotation:
```
$apt-get install imagemagick
```

### Install FFMPEG for transform fuckup to mp4:
```
$apt-get install ffmpeg
```



## VPN AND REMOTE SSH ROOT CONTROL
```
$apt-get install network-manager-vpnc
$apt-get install network-manager-vpnc-gnome
$apt-get install network-manager-pptp network-manager-pptp-gnome
```

### Open network manager and create VPN connection:
```
Add: PPTP
----------------
Connection name: YOURVPN
Gateway: your-domain-or-ip.nl
User name: youruser
Password: yourpassword
NT domain: <empty>
-----------------
IPV4 settings: 
Method: automatic
-----------------
PPTP advanced options:
Check AUTH only:
	MSCHAP
	MSCHAPv2

Security and compression:
ONLY Check:
Use point-to-point encryption
Allow BSD data compression
Allow deflate data compression
Use TCP header compression
```


## AutoUP VPN connection:
```
$pico /etc/NetworkManager/dispatcher.d/YOURVPN
```
Add:
```
#!/bin/bash
VPN_CONNECTION_NAME=”YOURVPN”
	sleep “3”
	nmcli con up id “${VPN_CONNECTION_NAME}”
```

## Reboot service:
```
$systemctl restart NetworkManager
```

## Autostart with VPN password:
```
$pico /etc/NetworkManager/system-connections/YOURVPN
```

```
[connection]
id=YOURVPN
uuid=27b2c31a-27b3-4969-9905-166611722df6
type=vpn
autoconnect=false
permissions=
timestamp=1549032424

[vpn]
gateway=your-domain-or-IP.nl
password-flags=0
require-mppe=yes
user=youruser
service-type=org.freedesktop.NetworkManager.pptp

[vpn-secrets]
password=yourpassword

[ipv4]
dns-search=
method=auto

[ipv6]
addr-gen-mode=stable-privacy
dns-search=
ip6-privacy=0
method=auto
```

## Auto reconnect VPN when wired connection is up:
```
$nm-connection-editor
```
### Ethernet > wired connection > edit > general > check “Automatically connect to VPN when using this connection

## EXTRA:
```
$nmcli c show
```
Copy UUID: MBVPN 09ea20b7-3732-4196-ba99-74a41e9e0ca7  vpn enp5s0
```
$pico /etc/init.d/autovpn.sh
```

```
#!/bin/bash
# A script which tries to establish VPN connection at every 20 seconds.

# Keep a log.
exec > >(tee -i ~/.autovpn.log)
exec 2>&1

# Enter all your VPN servers' UUID values here.
# Use "nmcli c show --active" command to learn UUID's of active VPN connections.
# They will be used in the order given here.
# You may enter as many servers as you like. More the better.
vpn=("09ea20b7-3732-4196-ba99-74a41e9e0ca7")

# Optional: You may enter server locations or names here.
# They are used in log file and command output.
vpn_name=("Netherlands" "Switzerland" "United Kingdom")

# Keep the number of alternatives in memory.
vpn_count=${#vpn[@]}

# This defines maximum attempt number to prevent infinite connection failure.
# Connection failure to all defined VPN servers counts only 1 attempt in this regard.
max_attempt=20

# Implemantation begins.

printf "\nVPN auto connection script started at $(date +"%F %T").\nIts log file is located at ~/autovpn.log\nIt will only report when no connection is found.\n"

for (( m=1; m<$max_attempt; m++ ));
	do
 	#printf "\n$(date +"%F %T") Waiting for 10 seconds before VPN connection check.\n"
	sleep 10
	is_connected=$( nmcli c show --active | grep tun0 )
 	if [ "$is_connected" = "" ];
    	then
    	printf "\n$(date +"%F %T") : VPN connection is not active!\n"
    	printf "$vpn_count VPN servers will be reached in given order to make a VPN connection.\n"
    	printf "If none of them works this whole process will start over for $(($max_attempt-$m)) times.\n"

    	for (( i=0; i<${vpn_count}; i++ ));
        	do
        	printf "\n*** Connecting to "${vpn_name[$i]}"... ***\n"
         	nmcli c up ${vpn[$i]}
         	if [ $? -eq 0 ];
             	then
             	printf "\n*** Connected to "${vpn_name[$i]}"! ***\n\nIt will be periodically checked to see if it is still connected.\n"
            	break
             	fi
         	done
      	else
     	#printf "$(date +"%F %T") Check result: VPN is already connected.\n"
     	m=$((m - 1))
     	fi
     	#printf "$(date +"%F %T") Loop is ending. Waiting for 10 seconds.\n"
 	sleep 10
 	Done
```

## Set to execute
```
$chmod +x /etc/init.d/autovpn.sh
$chmod 755 /etc/init.d/autovpn.sh
```



## Install openssh server:
```
$apt-get install openssh-server
$pico /etc/ssh/sshd_config
```
### Add: PermitRootLogin yes
```
$systemctl restart ssh
```

## Auto-hide panel 1 and panel 2!
### (use desktop configure)


## AUDIO LOOP: Install VLC if needed:
```
$apt-get install vlc
```
### Then add startup command to “Applications > Settings > Session and Startup” Tab: Application Autostart.

## START AUDIO LOOP ON BOOT CLI:
```
cvlc /home/youruser/vans_loop.mp3 --loop
```


## CLOUD SERVER:

### LINUX SERVER - FOR UPLOAD PURPOSES
```
sudo nano /etc/php5/apache2/php.ini OR sudo nano /etc/php/7.0/apache2/php.ini (LIVE)
Upload_max_size : 18mb
Max_size : 18mb
```



------------------------------------------------------------------------


### REMOTE SUPPORT
## LIVE SUPPORT CHECKS (SSH ROOT THROUGH VPN ONLY!)

### Set up VPN to MB internal network:
```
Type: L2TP over IPSec
Server: yourdomainorip.com
User: youruser
Pass: yourpassword
Shared secret: yourpassword
```

------------------------------------------------------------------------

## Logs/Tracing:

### Node logs:
```
$pico /var/log/yourservice.log
```

### Live read:
```
$tail -f /var/log/yourservice.log
```

### Restart/Status nodeJS service:
```
$service yourservice restart
$service yourservice status
```

## Extra configs (forced):
### Manual forced Xcfe4 monitor config for HD screens:
```
$pico /home/youruser/.config/xfce4/xfconf/xfce-perchannel-xml/displays.xml
```

```
<?xml version="1.0" encoding="UTF-8"?>

<channel name="displays" version="1.0">
  <property name="Default" type="empty">
    <property name="DVI-D-1" type="string" value="1. Samsung 48&quot;">
      <property name="Active" type="bool" value="true"/>
      <property name="Resolution" type="string" value="1920x1080"/>
      <property name="RefreshRate" type="double" value="60.000000"/>
      <property name="Rotation" type="int" value="0"/>
      <property name="Reflection" type="string" value="0"/>
      <property name="Primary" type="bool" value="true"/>
      <property name="Position" type="empty">
        <property name="X" type="int" value="0"/>
        <property name="Y" type="int" value="0"/>
      </property>
    </property>
    <property name="HDMI-2" type="string" value="2. Samsung 48&quot;">
      <property name="Active" type="bool" value="true"/>
      <property name="Resolution" type="string" value="1920x1080"/>
      <property name="RefreshRate" type="double" value="60.000000"/>
      <property name="Rotation" type="int" value="0"/>
      <property name="Reflection" type="string" value="0"/>
      <property name="Primary" type="bool" value="false"/>
      <property name="Position" type="empty">
        <property name="X" type="int" value="1920"/>
        <property name="Y" type="int" value="0"/>
      </property>
    </property>
  </property>
  <property name="Notify" type="bool" value="true"/>
</channel>
```




## Xcfe4 Startup configs (Chromium, audio etc):
### Location: /home/youruser/.config/autostart/
```
$pico /home/youruser/.config/autostart/START\ CHROMIUM\ LEFT-1.desktop
$pico /home/youruser/.config/autostart/START\ CHROMIUM\ RIGHT\(TOP\).desktop
```

