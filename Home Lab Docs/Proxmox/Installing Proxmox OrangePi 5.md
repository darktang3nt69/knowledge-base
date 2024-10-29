1. Install Armbian to SD Card.
2. Then login using `root` and `1234` as password.
3. Then run the below command:
    ```bash
    sudo armbian-config
    ```
4. Select `System` and then `Install` and then `Boot from MTD-flash`
5. Download: https://armbian.hosthatch.com/dl/orangepi5/archive/Armbian_24.8.1_Orangepi5_bookworm_current_6.10.6.img.xz
6. ```bash
    sudo dd bs=1M if=Armbian_24.8.1_Orangepi5_bookworm_current_6.10.6.img of=/dev/nvme0n1 status=progress
    sudo sync
    ```
6. Remove SD CARD and boot Normally.
7. Follow this guide: https://github.com/jiangcuo/Proxmox-Port/wiki/Install-Proxmox-VE-on-Debian-bookworm
8. [Configure Linux Bridge](obsidian://open?vault=knowledge-base&file=Home%20Lab%20Docs%2FProxmox%2FSetup%20Proxmox%20Bridge)
9. Optional: [Disable IPv6](obsidian://open?vault=knowledge-base&file=Home%20Lab%20Docs%2FProxmox%2FDisable%20IPv6)
10. https://codingfield.com/blog/2024-01/install-armbian-and-proxmox-on-orangepi5plus/