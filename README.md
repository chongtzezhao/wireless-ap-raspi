Connect to wifi internet

sudo nano /etc/netplan/50-cloud-init.yaml

```network:
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
        "mallab-Precision-7820-Tower":
          password: "YowLEMSw"
```
		  
sudo netplan --debug apply

networkctl
3 wlan0 wlan routable configured

sudo apt update
sudo apt install hostapd dnsmasq

CACHE LOCK
sudo lsof /var/lib/dpkg/lock
find PID and kill
sudo kill -9 $PID


HOSTAPD - if "Job for hostapd service failed", then kill systemd

sudo nano /etc/hostapd/hostapd.conf

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


sudo nano /etc/default/hostapd
DAEMON_CONF="/etc/hostapd/hostapd.conf"	

sudo systemctl unmask hostapd
sudo systemctl enable hostapd
sudo systemctl start hostapd


sudo apt-get install dnsmasq

ERROR: failed to create listening socket for port 53: Address already in use
FIX: sudo ss -lp "sport = :domain"
sudo systemctl disable systemd-resolved
sudo systemctl mask systemd-resolved

sudo cp /etc/dnsmasq.conf /etc/dnsmasq.conf.old
sudo nano /etc/dnsmasq.conf
interface=wlan0
dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h


sudo systemctl start dnsmasq

ERROR: unable to resolve host ubuntu: Temporary failure in name resolution
FIX:
sudo nano /etc/hosts
add: 127.0.0.1 $hostname
hostname can be found in cat /etc/hostname


sudo nano /lib/systemd/system/dnsmasq.service
[Unit]
...
After=network-online.target
Wants=network-online.target

    wlan0:
      dhcp4: false
      addresses:
      - 192.168.4.1/24
	  
	  
sudo reboot

ERROR: directory /etc/resolv.conf for resolv-file is missing, cannot poll
FAILED to start up
FIX: sudo rm -f /etc/resolv.conf
sudo nano /etc/resolv.conf
nameserver 8.8.8.8




https://askubuntu.com/questions/91543/apt-get-update-fails-to-fetch-files-temporary-failure-resolving-error#91595
