1. Up an interface in Linux (Temporary changes):
    ```bash
    sudo ip link set dev enp0s8 up
    # Explanation: set device interface_name up 
    ```
2. Add IPv4/IPv6 Address to an interface (Temporary changes):
    ```bash
    sudo ip address add [IPv4 or IPv6 cidr notation] dev enp0s8
    ```
3. Remove an IP address (Temporary changes):
    ```bash
    sudo ip address delete [IP Addr] dev enp0s8
    ```
4. Make Changes to this file to make any network related setting persist:
    ```bash
    sudo nano /etc/netplan/whatever_plan
    # Then apply the changes:
    sudo netplan try # Dry Run for the changes. Press Enter to accept the changes. ctrl + c to cancel. Default timeout: 2 mins.
    sudo netplan try --timeout 60 # set timeout to 60 secs.
    
    ```
