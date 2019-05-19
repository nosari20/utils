# Raspberry Pi hotpost with firewall

## Requirements

* Raspberry Pi 3 or above (Wi-Fi interface required)
* SD card (>1GB)

## Installation 

1. Download [Raspbian](https://www.raspberrypi.org/downloads/raspbian/)
2. Flash the img file to the SD card (with [Etcher](https://www.balena.io/etcher/) for example)
3. Unplug and replug the SD card to the computer and create file named ``ssh`` at the root of the SD card in order to enable SSH (keep the file empty)
4. Boot the RPi

## Setup

### General

1. Connect to the RPi with ``ssh`` (by default the login is ``pi`` and the password is ``raspberry``)
```sh
ssh pi@<RPI_IP>
 ```
2. Execute the following commands
```sh
sudo apt get update
sudo apt get upgrade
 ```
 3. Install hostapd and dnsmasq
 ```sh
sudo apt install hostapd
sudo apt install dnsmasq
 ```
 4. Stop services
 ```sh
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
sudo systemctl stop hostapd
sudo systemctl stop dnsmasq
```
5. Configure static ip for wlan0
```sh
sudo nano /etc/dhcpcd.conf
```
add the following lines at the en of the file
```
interface wlan0
static ip_address=192.168.0.10/24
denyinterfaces eth0
denyinterfaces wlan0
```

### DHCP configuration
1. Create the configuration file
```sh
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.original #copy the original file
sudo nano /etc/dnsmasq.conf
```
2. Put the following lines in the file
```
interface=wlan0
  dhcp-range=192.168.0.11,192.168.0.30,255.255.255.0,24h
```
### Acess Point configuration

1. Creat the configuration file
```sh
sudo nano /etc/hostapd/hostapd.conf
```
2. Put the following lines in the file
```
interface=wlan0
bridge=br0
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
ssid=NETWORK
wpa_passphrase=PASSWORD
```
3. Set default configuration
```sh
sudo nano /etc/default/hostapd
```
4. Replace the following line
```
#DAEMON_CONF=""
```
by
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```
### Allow traffic forwarding

1. Open the ```/etc/sysctl.conf`` file
```sh
sudo nano /etc/sysctl.conf
```
2. Delete the ``#`` at the begining of the following line 
```
#net.ipv4.ip_forward=1
```
### iptables rules

1. Add the following rule
```sh
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
``` 
2. Save the rule
```sh
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"

```
3. Add the following file at the end of the file ``/etc/rc.local``
```sh
sudo nano /etc/rc.local
```
```sh
iptables-restore < /etc/iptables.ipv4.nat # Add this line
```

### Create bridge

1. Install bridge-utils
```sh
 sudo apt-get install bridge-utils
 ```
 2. Ceate new bridge
 ```sh
 sudo brctl addbr br0
 sudo brctl addif br0 eth0
 ```
 3. Open the ``/etc/network/interfaces`` file
```sh
sudo nano /etc/network/interfaces
```
4. Add the following lines
```
auto br0
iface br0 inet manual
bridge_ports eth0 wlan0
```


### Reboot
```sh
sudo reboot
```

### Setup firewall

1. Install the web admin console
```sh
sudo apt-get install perl libnet-ssleay-perl openssl libauthen-pam-perl libpam-runtime libio-pty-perl apt-show-versions python
cd /tmp
wget http://prdownloads.sourceforge.net/webadmin/webmin_1.910_all.deb # latest version here
sudo dpkg --install webmin_1.910_all.deb
```
2. Enable bridge firewalling
```sh
modprobe br_netfilter
```
3. Login to ``https://<RPI_IP>:10000`` (same username and password as the ssh user)
4. Go to Networking > Linux Firewall and create forward rules (packet filtering)
```sh
1. Drop     If input physical interface is wlan0 # set default rule to drop (warning : the default action must be set to Accept and this rule must be the last)
2. Accept   If protocol is UDP and destination port is 53 # Allow DNS queries
3. Accept	If protocol is UDP and destination port is 67:68 # Allow DHCP

```





