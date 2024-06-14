1. [Install packages](https://phoenixnap.com/kb/ubuntu-samba):
```bash
sudo apt update
sudo apt install samba -y
whereis samba
samba -V
systemctl status smbd
```

2. Create a sharing directory:
```bash
sudo mkdir -p /home/sharing
```

3. Edit samba config file:
```bash
sudo vim /etc/samba/smb.conf
```

4. Add the config:
```bash
[sharing]
    comment = Samba share directory
    path = /home/sharing
    read only = no
    writable = yes
    browseable = yes
    guest ok = no
    valid users = @username
```

5. Allow Firewall and restart sambaz:
```bash
sudo ufw allow samba
sudo systemctl restart smbd
```