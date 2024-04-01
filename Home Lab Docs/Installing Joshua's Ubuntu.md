https://github.com/Joshua-Riek/ubuntu-rockchip/wiki/Orange-Pi-5

## [Install the bootloader to the SPI NOR flash](https://github.com/Joshua-Riek/ubuntu-rockchip/wiki/Orange-Pi-5#install-the-bootloader-to-the-spi-nor-flash)

Booting directly from a USB or NVMe requires flashing U-Boot to the SPI.

1. Boot from an SD Card.
2. Write the bootloader to the SPI NOR flash.

```
sudo dd if=/usr/lib/u-boot/rkspi_loader.img of=/dev/mtdblock0 conv=notrunc
sync
```

## [Install Ubuntu onto your NVMe from Linux](https://github.com/Joshua-Riek/ubuntu-rockchip/wiki/Orange-Pi-5#install-ubuntu-onto-your-nvme-from-linux)

1. Insert an NVMe SSD.
2. Boot from an SD Card.
3. Download the latest Orange Pi 5 image from GitHub.
4. Check if your NVMe SSD is detected.

```
ubuntu@ubuntu-desktop:~$ lsblk /dev/nvme0n1
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nvme0n1     259:0    0 119.2G  0 disk 
```

5. Write the downloaded Orange Pi 5 image to your NVMe drive.

```
xz -dc ubuntu-22.04.3-preinstalled-desktop-arm64-orangepi-5.img.xz | sudo dd of=/dev/nvme0n1 bs=4k status=progress; sudo sync
```