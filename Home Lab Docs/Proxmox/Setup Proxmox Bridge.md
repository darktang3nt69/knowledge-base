In Proxmox, when a physical interface is added to a bridge, IP addressing should only be configured on the bridge itself (`vmbr0`), not on the underlying interface (`end0`).

To correct this:

1. **Configure the IP Address on `vmbr0` Only**: Edit the network configuration file (usually located at `/etc/network/interfaces`), so `end0` acts solely as a bridge member, while `vmbr0` handles the IP configuration.

```bash
auto lo 
iface lo inet loopback 

# Configure end0 as bridge-only 
auto end0 
iface end0 inet manual 

# Configure vmbr0 with IP settings 
auto vmbr0 
iface vmbr0 inet static 
    address 192.168.1.69/24 
    gateway 192.168.1.1 
    bridge-ports end0 
    bridge-stp off 
    bridge-fd 0 
    dns-nameservers 8.8.8.8 8.8.4.4
```

2. **Apply the Configuration**: Restart Orangepi5