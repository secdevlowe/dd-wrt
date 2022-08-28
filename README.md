# DD-WRT Default Configuration

My DD-WRT configuration used on my home network. This is documentation for myself. I want to make sure I have my preferred configuration documented when I need to update/reset this box.

## Router 

* [Netgear R7600v3](https://www.netgear.com/support/product/R6700v3.aspx).
* DD-WRT v3.0-r48646 std (04/12/22)
* Kernel VersionLinux 4.4.302 #5712 SMP Tue Apr 12 04:34:21 +07 2022 armv7l

## DD-WRT Build

* [v3.0-r47090 std (07/26/21)](https://dd-wrt.com/support/router-database/?model=R6700_v1)

## 3rd-Party Services

* [ProtonVPN](https://protonvpn.com/)
* [NextDNS](https://nextdns.io/)

## Optimal ProtonVPN Server Autorun

* Automated [script](https://gist.github.com/secdevlowe/c3ae773b9c2a45d91d7df59ba07b5d35) using ProtonVPN's API for finding optimal VPN server connection.
* All credit and a huge thank you goes to [collinbarret](https://github.com/collinbarrett) for posting instructions in his [blog](https://collinmbarrett.com/protonvpn-dd-wrt-api-script/).
* View the [Entware Install & Automation Script.md](https://github.com/secdevlowe/dd-wrt/blob/main/Entware%20Install%20%26%20Automation%20Script.md) file for instructions on setting this up.

# Setup

## Basic Setup

### WAN Setup

#### WAN Connection Type

* Connection Type: Automatic Configuration - DHCP
* Ignore WAN DNS: `☑`

## Optional Settings

* Router Name: {REDACTED}
* Hostname: {REDACTED}
* Domain Name: {REDACTED}

## Network Setup

#### Router IP

__Change default local IP address for (slightly) better network [security]:(https://routersecurity.org/ipaddresses.php)__
__Mirror the local IP address to the gateway IP. This fixed my issue with my Xfinity Xfi Modem injecting DNS via DHCP for my Windows PC DNS dynamic update feature - (trustedsec)[https://www.trustedsec.com/blog/injecting-rogue-dns-records-using-dhcp/] and (netspi):[https://www.trustedsec.com/blog/injecting-rogue-dns-records-using-dhcp/]__

* Local IP Address: 192.168.30.11/24
* Gateway: 192.168.30.11
* Local DNS: 0.0.0.0

#### Network Address Server Settings (DHCP)

__Route DNS to private network reserved IPs to ensure ISP's DNS servers are not used. Dnsmasq is used to configure preferred DNS servers.__
__Local DNS should be set to 0.0.0.0 - forcing the nextdns.io__

* DHCP Type: DHCP server
* DHCP Server: [x] Enable
* Start IP Address: 192.168.1.64
* Maximum DHCP Users: 64
* Client Lease Expriation: 1440
* Static DNS 1: `10.0.0.0`
* Static DNS 2: `10.0.0.1`
* Static DNS 3: `10.0.0.2`
* Use DNSMasq for DNS: `☑`
* DHCP-Authoritative: `☑`
* Recursive DNS Resolving (Unbound): `☐`
* Forced DNS Redirection: `☑`

#### Time Settings

* Time Zone: `US/Mountain`
* Server IP/Name: `pool.ntp.org`

#### Dnsmasq

* No DNS Rebind: `☑`
* Query DNS in Strict Order: `☑`
* Maximum Cached Entries: `10000`

# Wireless

## Basic Settings

Change SSID(s), turn on WPA2-PSK, and set a strong password(s) for network authentication.

Use guidance from [here](https://forum.dd-wrt.com/wiki/index.php/QCA_wireless_settings) for additional wireless settings available for Qualcomm Atheros (QCA) 802.11a/b/n/ac/ad routers.

# Services

## Services Management

### Dnsmasq 

* DNSmasq: `☑` Enable
* No DNS Rebind: `☑` Enable
* Query DNS in Strict Order: `☑` Enable

__Additional Dnsmasq Options__

```
# Block Comcast WPAD / DNS
address=/comcast.net/

# Block non-domain lookups.
domain-needed

# Override default synchronous logging which blocks subsequent requests.
log-async=5

# https://github.com/collinbarrett/dd-wrt/issues/1
neg-ttl=300

# Use fastest response from any upstream.
all-servers

# Use nextdns.io for DNS.
no-resolv
bogus-priv
server=45.90.30.0
server=45.90.28.0
add-cpe-id={REDACTED}
```

## VPN

### OpenVPN Client

Configure using the [latest guidance from ProtonVPN](https://protonvpn.com/support/vpn-router-ddwrt/).

__Additional Config__

```
...
# Additional settings from ProtonVPN Documentation https://protonvpn.com/support/vpn-router-ddwrt/
tls-client
remote-cert-tls server
remote-random
nobind
tun-mtu 1500
tun-mtu-extra 32
mssfix 1450
persist-key
persist-tun
ping-timer-rem
reneg-sec 0

# Route DNS requests from dnsmasq through OpenVPN client.
route 45.90.28.0 255.255.255.255
route 45.90.30.0 255.255.255.255
```

# Security

## Firewall

### Security

#### Firewall Protection

* SPI Firewall: `Enable`

#### Additional Filters

* Filter Proxy: `☑`
* Filter Cookies: `☑`
* Filter Java Applets: `☑`
* Filter ActiveX: `☑`
* Filter TOS/DSCP: `☑`
* ARP Spoofing Protection: `☑`

#### Block WAN Requests

* Block Anonymous WAN Requests (ping): `☑`
* Filter Multicast: `☑`
* Filter WAN NAT Redirection: `☑`
* Filter IDENT (Port 113): `☑`
* Block WAN SNMP access: `☑`

#### Impede WAN DoS/Bruteforce

* Limit SSH Access: `☑`
* Limit Telnet Access: `☑`
* Limit PPTP Server Access: `☑`
* Limit FTP Server Access: `☑`

## VPN Passthrough

### Vitual Private Network (VPN)

#### VPN Passthrough

* IPSec Passthrough: `Disable`
* PPTP Passthrough: `Disable`
* L2TP Passthrough: `Disable`

# Administration

## Router Management

### Web Access

#### Protocol: `☑` HTTPS

## Keep Alive

### WDS/Connection Watchdog

_If WAN connectivity is lost (either VPN or ISP connection break), reboot the router. If it is an ISP issue, this likely will not help. If the VPN server I was connected to goes down, rebooting the router will re-connect to a new server._

* Enable Watchdog: `Enable`
* Interval (in seconds): `60`
* IP Addresses: `1.1.1.1`

## Commands

### Diagnostics

#### Startup

```
# Start Entware.
# https://wiki.dd-wrt.com/wiki/index.php/Installing_Entware
sleep 25
/opt/etc/init.d/rc.unslung start
```

#### Firewall

```
LAN_IP=`nvram get lan_ipaddr`
WAN_IF=`nvram get wan_iface`

# -I w/o specified rulenum => specify in reverse priority

# block non-VPN requests
iptables -I FORWARD -i br0 -o $WAN_IF -j REJECT --reject-with icmp-host-prohibited
iptables -I FORWARD -i br0 -p tcp -o $WAN_IF -j REJECT --reject-with tcp-reset
iptables -I FORWARD -i br0 -p udp -o $WAN_IF -j REJECT --reject-with udp-reset

# allow TV to bypass VPN
iptables -I FORWARD -i br0 -s 192.168.1.63 -o $WAN_IF -j ACCEPT

# block non-VPN DNS requests
# TODO: https://github.com/collinbarrett/dd-wrt/issues/3
# iptables -I FORWARD -o $WAN_IF -p tcp --dport 53 -j REJECT --reject-with tcp-reset
# iptables -I FORWARD -o $WAN_IF -p udp --dport 53 -j REJECT --reject-with udp-reset
# iptables -I OUTPUT -o $WAN_IF -p tcp --dport 53 -j REJECT --reject-with tcp-reset
# iptables -I OUTPUT -o $WAN_IF -p udp --dport 53 -j REJECT --reject-with udp-reset

# redirect DNS requests to dnsmasq
iptables -t nat -I PREROUTING -i br0 -p tcp --dport 53 -j DNAT --to $LAN_IP
iptables -t nat -I PREROUTING -i br0 -p udp --dport 53 -j DNAT --to $LAN_IP
```
