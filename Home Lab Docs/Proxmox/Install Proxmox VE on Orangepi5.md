

## Install a standard Debian bookworm (arm64)
### Add an /etc/hosts entry for your IP address
Please make sure that your machine's hostname is resolvable via `/etc/hosts`, i.e. you need an entry in `/etc/hosts` which assigns an address to its hostname.

Make sure that you have configured one of the following addresses in /etc/hosts for your hostname:

- 1 IPv4 or
- 1 IPv6 or
- 1 IPv4 and 1 IPv6

Note: This also means removing the address 127.0.1.1 that might be present as default.

For instance, if your IP address is `192.168.15.77`, and your hostname `prox4m1`, then your `/etc/hosts` file could look like:

```
127.0.0.1       localhost.localdomain localhost
192.168.15.77   prox4m1.proxmox.com prox4m1

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

You can test if your setup is ok using the hostname command:

```
hostname --ip-address
192.168.15.77 # should return your IP address here
```

## Install Proxmox VE

### Add the Proxmox VE repository:

```
echo 'deb [arch=arm64] https://mirrors.apqa.cn/proxmox/debian/pve bookworm port'>/etc/apt/sources.list.d/pveport.list
```

Add the Proxmox VE repository key:

```
curl -L https://mirrors.apqa.cn/proxmox/debian/pveport.gpg -o /etc/apt/trusted.gpg.d/pveport.gpg 
```

Update your repository and system by running:

```
apt update && apt full-upgrade -y
```

### Install Proxmox VE packages

Install the ifupdown2 packages

```
rm -rf /tmp/.ifupdown2-first-install && apt install ifupdown2 -y
```

Install the Proxmox VE packages

```
apt install proxmox-ve postfix open-iscsi -y
```

Configure packages which require user input on installation according to your needs (e.g. Samba asking about WINS/DHCP support). If you have a mail server in your network, you should configure postfix as a satellite system, your existing mail server will then be the relay host which will route the emails sent by the Proxmox server to their final recipient.

If you don't know what to enter here, choose local only and leave the system name as is.

Finally, you can connect to the admin web interface ([https://youripaddress:8006](https://youripaddress:8006/)).

## [Rockchip note](https://github.com/jiangcuo/Proxmox-Port/wiki/Install-Proxmox-VE-on-Debian-bookworm#rockchip-note)
### [All rockchip soc are not compatible with the mainline OVMF.](https://github.com/jiangcuo/Proxmox-Port/wiki/Install-Proxmox-VE-on-Debian-bookworm#all-rockchip-soc-are-not-compatible-with-the-mainline-ovmf)

If proxmox ve < 8.1, do

```shell
apt download pve-edk2-firmware=3.20220526-1
dpkg -i pve-edk2-firmware_3.20220526-1_all.deb
```

If proxmox ve >= 8.1 do

```shell
apt download pve-edk2-firmware-aarch64=3.20220526-rockchip
dpkg -i pve-edk2-firmware-aarch64_3.20220526-rockchip_all.deb
```

### big.LITTLE !

[](https://github.com/jiangcuo/Proxmox-Port/wiki/Install-Proxmox-VE-on-Debian-bookworm#biglittle-)

For example, RK3389 RK3588 is a big.little architecture. You have to configure cpu-affinity to make sure that vm use big cores only or little cores only when vcpus > 1. see [https://github.com/jiangcuo/Proxmox-Arm64/issues/28](https://github.com/jiangcuo/Proxmox-Arm64/issues/28)

## [Troubleshooting](<(https://github.com/jiangcuo/Proxmox-Port/wiki/Install-Proxmox-VE-on-Debian-bookworm#troubleshooting)>)

### [resolv.conf gets overwritten](<(https://github.com/jiangcuo/Proxmox-Port/wiki/Install-Proxmox-VE-on-Debian-bookworm#resolvconf-gets-overwritten)>)

The PVE GUI expects to control DNS management and will no longer take its DNS settings from /etc/network/interfaces. Any package that auto-generates (overwrites) /etc/resolv.conf will cause DNS to fail, e.g. packages 'resolvconf' for IPv4 and 'rdnssd' for IPv6.

### [ipcc_send_rec[1] failed](<(https://github.com/jiangcuo/Proxmox-Port/wiki/Install-Proxmox-VE-on-Debian-bookworm#ipcc_send_rec1-failed)>)

```
ipcc_send_rec[1] failed: Connection refused
```

then you should review your /etc/hosts file according to the instructions above.

[https://pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_12_Bookworm](https://pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_12_Bookworm)