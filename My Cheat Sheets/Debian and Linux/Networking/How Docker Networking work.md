Docker first creates a virtual bridge on the host as `docker0`. This bridge acts as an interface to the host and as a `router/switch` for the containers network namespaces.

What actually happens when a container is run in the context of networking namespace:

1. Creates a Network Namespace for a container. 
2. Creates an interface in the container namespace.
3. Created a VETH pairs. 
4. One end is connected to the interface(in the networking namespace) created above and the other to the `docker0` bridge.
5. Assigns the IP.
6. Bring the interfaces UP.
7. Enable NAT - IP Masquerade. This is port mapping between the host port to the network namespace port. With the help of iptables, a rule is added to forward the traffic in docker chain.

We can imitate docker networking using the below commands:

1. Create a networking namespace: 
    ```bash
    sudo ip netns add red # replace red with desired name
    ```
2. Create a new bridge:
    ```bash
    ip link add v-net-0 type bridge # replace v-net-0 with desired name
    ```
3. Up the bridge interface`v-net-0`:
    ```bash
    ip link set dev v-net-0 up
    ```
4. Assign an IP to the bridge interface:
    ```bash
    ip addr add 192.168.15.5/24 dev v-net-0
    ```
5. Create VETH pairs:
    ```bash
    ip link add veth-red type veth peer name veth-red-br
    ```
6. Add `veth-red` interface to `red` networking namespace:
    ```bash
    ip link set veth-red netns red
    ```
7. Assign an IP to the above interface `veth-red`  attached to `red`  networking namespace:
    ```bash
    ip -n red addr add 192.168.15.1 dev veth-red
    ```
8. Up `veth-red`:
    ```bash
    ip -n red link set veth-red up
    ```
9. Attach `veth-red-br` to `v-net-0`: 
    ```bash
    ip link set veth-red-br master v-net-0
    ```
10. 