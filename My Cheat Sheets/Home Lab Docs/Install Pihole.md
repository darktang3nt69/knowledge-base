1. Save this manifest to ==pihole.yaml== : 
```yaml
version: "3.9"
services:
  pihole:
    container_name: pihole
    image:  pihole/pihole:latest
    pull_policy: always
    environment:
      TZ: 'Asia/Kolkata'
      WEBPASSWORD: '7799504208aA@' # set your password
      ServerIP: 192.168.1.8 # make sure this ip is within the subnet of the macvlan below
    volumes:
      - '~/pihole-config:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'
    networks:
      pihole_net:
        ipv4_address: 192.168.1.8
    cap_add:
      - NET_ADMIN
    restart: unless-stopped
networks:
  pihole_net:
    driver: macvlan
    driver_opts:
      parent: eth0
    ipam:
      config:
        - subnet: 192.168.1.0/24
```

2. Apply the above docker compose file:
```bash
docker compose -f pihole.yaml up -d
```