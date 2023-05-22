1. Always update the server before installing any package.
```bash
sudo apt update 
sudo apt upgrade
```

2. Install the packages.
```bash
sudo apt install samba smbclient cifs-utils
```

3. open the following ports.
```bash
sudo ufw allow 139
sudo ufw allow 445
```

4. Create the samba share folder.
```bash
mkdir /sambashare
```

5. Before editing, take backup of `samba.conf` file.
```bash
sudo cp /etc/samba/smb.conf smb.conf.orig
```

6. Open `/etc/samba/smb.conf`, and add the following.
```bash
sudo /etc/samba/smb.conf
```
```
[samba_share]
comment = home server
path = /home/lokesh/sambashare
read-only = no
browsable = yes
valid users = smbuser lokesh
writeable=yes
force create mode = 770
force directory mode = 770
inherit acls = no
inherit permissions = yes
```

7. Add a samba user `smbuser`.
```bash
sudo useradd smbuser
```

8. Make `smbuser` the owner of the samba share.
```bash
chown smbuser:smbuser /smbshare/
```

9. Restart samba service.
```bash
sudo systemctl restart smbd
```