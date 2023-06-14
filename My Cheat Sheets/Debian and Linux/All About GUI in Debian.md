This guide describes how to install, enable and disable GUI on Debian

### Installation of GUI

1.  Run APT system update:

``` bash
sudo apt update && sudo apt upgrade
```

2.  Install Tasksel:

```bash
sudo apt install tasksel -y
```

3.  Install XFCE Desktop on Debian 11:

```bash
sudo tasksel install xfce-desktop
```

4. Set it as default gui:

```bash
sudo systemctl get-default
sudo systemctl set-default graphical.target
```


### Disable GUI

```bash
sudo systemctl set-default multi-user.target
sudo reboot
```

### Enable GUI

```bash
sudo systemctl set-default graphical.target
sudo reboot
```