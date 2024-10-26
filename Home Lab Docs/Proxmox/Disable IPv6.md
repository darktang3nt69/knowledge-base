If you don’t have the option to disable IPv6 directly in netplan or if your network configuration is managed differently, you can try these alternative methods to disable IPv6 on your system:

### 1. Disable IPv6 via `sysctl`

1. Edit the `sysctl.conf` file:
```bash
sudo nano /etc/sysctl.conf
```
2. Add the following lines to disable IPv6 for all network interfaces:
```bash
net.ipv6.conf.all.disable_ipv6 = 1 
net.ipv6.conf.default.disable_ipv6 = 1
```
    
3. Apply the changes:
```bash
sudo sysctl -p
```