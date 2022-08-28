__Again, all credit and a huge thank you goes to [collinbarret](https://github.com/collinbarrett) for posting instructions in his [blog](https://collinmbarrett.com/protonvpn-dd-wrt-api-script/).__

# Entware

Refer to the DD-WRT documentation [Installing Entware]:(https://wiki.dd-wrt.com/wiki/index.php/Installing_Entware) for additional help on the steps below.

## Preparation

1. Log in to the router's GUI, go to the USB page (Services - USB), Make sure Core USB Support, USB Storage and Automatic Drive Mount are all enabled. If one or more are not, enable them and click apply.
2. On your computer plug in the USB stick or harddrive, reformat the USB stick or harddrive using a program like Partition Wizard (Windows) or GParted (Linux). Make sure the format of the drive is ext2 for a USB stick, or ext3 or NTFS if it is a harddrive, Primary, not Logical. Label should be Optware if you want it to mount automatically.
3. (Optional) Install an [ext driver on Windows]:(http://www.ext2fsd.com/), helpful for troubleshooting.
4. Determine the architecture of the router you are using.
    * Check the Router Status page in the webUI
    > or
    * via telnet or ssh issue the command 'uname -a'
5. Make a note of this for later. If you can't determine the architecture to use, ask in the forums.
    _Note_: if the router is using a Linux 4.X kernel, Entware-3X may not work.

## Installation

1. Plug in the USB stick into the router. The router 'may' have to be rebooted. Check Services – USB to see if it shows up. Make a note of the current mount point, ex: /tmp/mnt/sda_part1, should be /opt if you did the above correctly.
3. Open up a putty (SSH) terminal to the router. Type in the following commands using the mount point above:

```sh
# Start Entware.
# https://wiki.dd-wrt.com/wiki/index.php/Installing_Entware
sleep 25
/opt/etc/init.d/rc.unslung start
```

   * NOTE: if you get nslookup: can't resolve 'bin.entware.net' then most likely your /etc/resolv.conf file has only a local nameserver entry (nameserver 192.168.1.1). Edit your /etc/resolv.conf file insert nameserver 8.8.8.8 save the file, and repeat your wget
  * The third command from above sh generic.sh runs the install script which downloaded several files from the internet and set up Entware. It will most likely show several packages that have no valid architecture, just ignore them. These will show up when any opkg command is run.

4. When installation is complete, run an update:

```sh
opkg update (click enter)
opkg upgrade (click enter)
```

5. The Final Step is to add the following commands to the start-up script (Administration tab - then Commands). The sleep value can be adjusted, but 10 is long enough for most USB Harddrives/routers:

```sh
sleep 10
/opt/etc/init.d/rc.unslung start
```

__Installation of Entware is now complete__

### Install jq

`opkg install jq`

# curl

```sh
curl -s -H "Cache-Control: no-cache" -H "Accept: application/json" https://api.protonmail.ch/vpn/logicals
```

This may require you to add  `-k (--insecure)` for permissions to run.

# jq

```sh
echo "$LOGICALS" | jq '.LogicalServers | map(select(.Status == 1 and .Tier == 2 and .Features == 8 and (.City | (contains("Atlanta") or contains("Dallas") or contains("Chicago"))))) | [sort_by(.Score, .Load)[]][0] | .Servers[0].EntryIP'
```

# Shell Script

```sh
#!/bin/sh

# specify PATH to run from cron
PATH=/bin:/usr/bin:/sbin:/usr/sbin:/jffs/sbin:/jffs/bin:/jffs/usr/sbin:/jffs/usr/bin:/mmc/sbin:/mmc/bin:/mmc/usr/sbin:/mmc/usr/bin:/opt/sbin:/opt/bin:/opt/usr/sbin:/opt/usr/bin

  on router startup, wait for network to initially connect
# sleep 25

# verify not on VPN when fetching ProtonVPN server scores
# MYPUBLICIP=$(curl -s http://whatismyip.akamai.com/)
# echo "My public IP is ${MYPUBLICIP}"

# fetch ProtonVPN server info
LOGICALS=$(curl -sk -H "Cache-Control: no-cache" -H "Accept: application/json" https://api.protonmail.ch/vpn/logicals)

# query for optimal server
IPSTRING=$(echo "$LOGICALS" | jq '.LogicalServers | map(select(.Status == 1 and .Tier == 2 and .Features == 8 and (.City | (contains("Atlanta") or contains("Dallas") or contains("Chicago"))))) | [sort_by(.Score, .Load)[]][0] | .Servers[0].EntryIP')

if [ -n "$IPSTRING" ]
then
  IPSTRING="${IPSTRING%\"}"
  IP="${IPSTRING#\"}"

  CURRENTIP=$(nvram get openvpncl_remoteip)
  echo "The current OpenVPN server IP is ${CURRENTIP}"

  if [ "$IP" != "$CURRENTIP" ]
  then
    echo "Connecting to $IP..."

    # update OpenVPN server
    nvram set openvpncl_remoteip="${IP}"
    nvram commit

    # restart openvpn
    stopservice openvpn
    startservice openvpn
  else
    echo "The optimal ProtonVPN server ($IP) has not changed. Aborting..."
  fi
else
  echo "Failed to fetch ProtonVPN server info. Aborting..."
fi
```

# Cron

dd-wrt login > Administration > Management > Cron

```sh
25 7 * * * root /opt/vpn-refresh.sh
0 17 * * * root /opt/vpn-refresh.sh
```

# Startup

WDS/Connection Watchdog is enabled in dd-wrt. Take a copy of the VPN refresh script and change the file extension from '.sh' to '.wanup'. Then move it under the '/jffs/etc/config/' directory: '/jffs/etc/config/vpn-refresh.wanup' .