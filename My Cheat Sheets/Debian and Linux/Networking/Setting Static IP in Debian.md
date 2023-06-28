## Setting Static IP in Debian

1. Find the Network Interface using:
```bash
ip -c link show
```

2. Open `etc/network/interfaces`  which contains the network config info.
```bash
sudo nano /etc/network/interfaces
```

3. Look for primary interface.
`Note enp7s0 means ethernet and wlp6s0 is wireless`
```
allow-hotplug enp0s5
iface enp0s5 inet dhcp
```

4. Comment DHCP and alllow-hotplug. and find these values.
- Get netmask from :
```bash
sudo ifconfig
```

 - get default from:
```bash
ip route | grep default
```


5. Subs the above into the below config and save the file.

```bash
auto enp7s0
iface enp7s0 inet static
address 192.168.1.69
netmask 255.255.255.0
gateway 192.168.1.1
dns-domain google.com
dns-nameservers 8.8.8.8
dns-nameservers 1.1.1.1
domain darktang3nt.local
```

6. Use the systemctl command as follows:  
```bash
sudo systemctl restart networking.service 
```

7. Make sure service restarted without any errors. Hence, type the following command:  
```bash 
sudo systemctl status networking.service
```

8. Current Config
```
auto lo
iface lo inet loopback


[[allow-hotplug]] enp7s0
[[iface]] enp7s0 inet dhcp

auto enp7s0
iface enp7s0 inet static
address 192.168.1.69
netmask 255.255.255.0
gateway 192.168.1.1
dns-domain google.com
dns-nameservers 8.8.8.8
dns-nameservers 1.1.1.1
domain darktang3nt.local

auto wlp6s0
iface wlp6s0 inet static
address 192.168.1.69
netmask 255.255.255.0
gateway 192.168.1.1
dns-domain google.com
dns-nameservers 8.8.8.8
dns-nameservers 1.1.1.1
domain darktang3nt.local
```

  
#### Sample session:

```
networking.service - Raise network interfaces
   Loaded: loaded (/lib/systemd/system/networking.service; enabled; vendor preset: enabled)
   Active: active (exited) since Wed 2021-01-27 23:10:00 IST; 1min 38s ago
     Docs: man:interfaces(5)
  Process: 1104 ExecStart=/sbin/ifup -a --read-environment (code=exited, status=0/SUCCESS)
 Main PID: 1104 (code=exited, status=0/SUCCESS)
 ```