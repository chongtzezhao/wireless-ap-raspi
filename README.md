This is the process for creating a wireless access point on a raspberry pi.

#### If you encounter errors at any point, please refer to the [section](#ERRORS) at the bottom

## Step 0: Connecting to Wifi
**Skip this if you have an ethernet connection**

We need to edit the network config file

```
sudo nano /etc/netplan/50-cloud-init.yaml
```

and type and this
```
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
      optional: true
  wifis:
    wlan0:
      dhcp4: true
      optional: true
      access-points:
        "YOUR WIFI SSID":
          password: "YOURWIFIPASSWORD"
```
connect to the wifi - 
`sudo netplan --debug apply`

check if the connection has been established using
`networkctl`


Output should look something like this:

```3 wlan0 wlan routable configured```

Note the keywords "routable" and "configured".

## Step 1: Installing the packages
```
sudo apt update
sudo apt install hostapd dnsmasq
```
### Step 1.1: hostapd
Edit/create the config file
`sudo nano /etc/hostapd/hostapd.conf`
and input the following
```
interface=wlan0
driver=nl80211
ssid=MyHotspot
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=12345678
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

**NOTE: driver is "nl..."("NL...") NOT "n1..."**

set the daemon for the hostapd service
```
sudo nano /etc/default/hostapd
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

restart hostapd

```
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
sudo systemctl start hostapd
```

### Step 1.2: dnsmasq

Create a backup of the file and edit the new one:

```
sudo cp /etc/dnsmasq.conf /etc/dnsmasq.conf.old
sudo nano /etc/dnsmasq.conf
```
inside /etc/dnsmasq.conf, add the following:

```
interface=wlan0
dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h
```

Next, start dnsmasq

`sudo systemctl start dnsmasq`

To ensure that dnsmasq starts properly on every time the rpi is rebooted, we need to change the service config file

`sudo nano /lib/systemd/system/dnsmasq.service`

inside:
```
[Unit]
...
After=network-online.target
Wants=network-online.target
```
remember `/etc/netplan/50-cloud-init.yaml`?

remove the "wifis" section and add the following under the "ethernets" section
```
    wlan0:
      dhcp4: false
      addresses:
      - 192.168.4.1/24
```
Make sure to indent it properly!

It should look something like this
```
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
      optional: true
    wlan0:
      dhcp4: false
      addresses:
      - 192.168.4.1/24
```
	  
sudo reboot

## ERRORS

### ERROR: CACHE LOCK
show running services

`sudo lsof /var/lib/dpkg/lock`


find PID and kill


`sudo kill -9 $PID`, where $PID is the PID of the service


HOSTAPD - if "Job for hostapd service failed", then kill systemd

### ERROR: failed to create listening socket for port 53: Address already in use
FIX: `sudo ss -lp "sport = :domain"`

the problem will most likely be systemd
```
sudo systemctl disable systemd-resolved
sudo systemctl mask systemd-resolved
```


### ERROR: unable to resolve host ubuntu: Temporary failure in name resolution
FIX:
```
sudo nano /etc/hosts
```
add `127.0.0.1 $hostname` into file.


hostname can be found using `cat /etc/hostname`


### ERROR: directory /etc/resolv.conf for resolv-file is missing, cannot poll
FAILED to start up

FIX: 
```
sudo rm -f /etc/resolv.conf
sudo nano /etc/resolv.conf
nameserver 8.8.8.8
```

Trivia: 8.8.8.8 is Google's DNS

https://askubuntu.com/questions/91543/apt-get-update-fails-to-fetch-files-temporary-failure-resolving-error#91595
