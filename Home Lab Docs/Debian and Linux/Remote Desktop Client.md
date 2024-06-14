This tutorial describes how to install and configure Xrdp server on Debian 10 Linux.

If you are looking for an open-source solution for remote desktop access, then you should check [VNC](https://linuxize.com/post/how-to-install-and-configure-vnc-on-debian-9/) .

## Installing Desktop Environment

Typically, Linux servers don’t have a desktop environment installed by default. The first step is to install X11 and a desktop environment that will act as a backend for Xrdp.

There are several desktop environments (DE) available in Debian repositories. We’ll be installing [Xfce](https://xfce.org/) . It is a fast, stable, and lightweight desktop environment, which makes it ideal for usage on a remote server. If you prefer another desktop environment like Gnome, you can install it instead of Xfce.

Enter the following commands as root or [user with sudo privileges](https://linuxize.com/post/how-to-create-a-sudo-user-on-debian/) to install Xfce on your server:

```
sudo apt update
```

Depending on your system and connection, downloading and installing Xfce packages will take some time.

## Installing Xrdp

Xrdp package is available in the standard Debian repositories. To install it, run:

```
sudo apt install xrdp 
```

The service will automatically start once the installation process is complete. You can verify that the Xrdp service is running by typing:

```
sudo systemctl status xrdp
```

The output will look something like this:

```output
● xrdp.service - xrdp daemon
   Loaded: loaded (/lib/systemd/system/xrdp.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2020-04-01 21:19:11 UTC; 4s ago
  ...
```

By default Xrdp uses the `/etc/ssl/private/ssl-cert-snakeoil.key` file which is readable only by users that are members of the “ssl-cert” group. Execute the following command to [add the `xrdp` user to the group](https://linuxize.com/post/how-to-add-user-to-group-in-linux/) :

```
sudo adduser xrdp ssl-cert  
```

That’s it. Xrdp has been installed on your Debian system.

## Configuring Xrdp

The Xrdp configuration files are stored in the `/etc/xrdp` directory. For basic Xrdp connections, you do not need to make any changes to the configuration files. Xrdp will use the default X Window desktop, which in this case, is XFCE.

The main configuration file is named [`xrdp.ini`](https://linux.die.net/man/5/xrdp.ini) . This file is divided into sections and allows you to set global configuration settings such as security and listening addresses and create different xrdp login sessions.

Whenever you make any changes to the configuration file you need to restart the Xrdp service:

```
sudo systemctl restart xrdp
```

Xrdp uses `startwm.sh` file to launch the X session. To use another X Window desktop, edit this file.

## Configuring Firewall

By default, Xrdp listens on port `3389` on all interfaces. If you run a firewall on your Debian server, which you should always do, you’ll need to add a rule that will enable traffic on the Xrdp port.

Assuming you use [ufw](https://linuxize.com/post/how-to-setup-a-firewall-with-ufw-on-debian-10/) to manage the firewall, run the following command to allow access to the Xrdp server from a specific IP address or IP range, in this example `192.168.1.0/24`:

```
sudo ufw allow from 192.168.1.0/24 to any port 3389
```

If you want to allow access from anywhere (which is highly discouraged for security reasons) run:

```
sudo ufw allow 3389
```

If you are using [nftables](https://wiki.debian.org/nftables) to filter connections to your system, open the necessary port by issuing the following command:

```
sudo nft add rule inet filter input tcp dport 3389 ct state new,established counter accept
```

For increased security, you may consider setting up Xrdp to listen only on localhost and creating an [SSH tunnel](https://linuxize.com/post/how-to-setup-ssh-tunneling/) that securely forwards traffic from your local machine on port `3389` to the server on the same port. Another secure option is to [install OpenVPN](https://linuxize.com/post/how-to-set-up-an-openvpn-server-on-debian-9/) and connect to the Xrdp server trough the private network.

## Connecting to the Xrdp Server

Now that you have set up your Xrdp server, it is time to open your Xrdp client and connect to the server.

If you have a Windows PC, you can use the default RDP client. Type “remote” in the Windows search bar and click on “Remote Desktop Connection”. This will open up the RDP client. In the “Computer” field, enter the remote server IP address and click “Connect”.

![RDP Client](https://linuxize.com/post/how-to-install-xrdp-on-debian-10/rdp-client_hu4ee2143b511ba1de0cac8a5cbe7d6882_48245_768x0_resize_q75_lanczos.jpg)

On the login screen, enter your [username](https://linuxize.com/post/how-to-add-and-delete-users-on-debian-9/) and password and click “OK”.

![RDP Login](https://linuxize.com/post/how-to-install-xrdp-on-debian-10/rdp-login_hu50726b7ceba78e54c787cf2f3b2050f4_34726_768x0_resize_q75_lanczos.jpg)

Once logged in, you should see the default Xfce desktop. It should look something like this:

![Xrdp XFCE Desktop](https://linuxize.com/post/how-to-install-xrdp-on-debian-10/xrdp-xfce-desktop_hu740a37e790d7a7dd468b63b4f22c7223_55728_768x0_resize_q75_lanczos.jpg)

You can now start interacting with the remote XFCE desktop from your local machine using your keyboard and mouse.

If you are using macOS, you can install the Microsoft Remote Desktop application from the Mac App Store. Linux users can use an RDP client such as Remmina or Vinagre.