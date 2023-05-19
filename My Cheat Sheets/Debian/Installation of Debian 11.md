## Installation of Debian 11
---

	The first step to installing Debian is to download the installation image for your machine, based on the architecture.

1. **Download**: [Debian](https://www.debian.org/releases/stable/)

2. Burning Debian ISO to USB

   - **Download**: **[Rufus](https://rufus.ie/en/)**.
   - Choose the ISO and burn the image.

3. Installing Debian

	- Boot into USB.
	- Choose Graphical Interface.
	- Choose Language.
	- Choose Country.
	- Choose Hostname(This will be the name of the server).
	- Enter Root Passwd.
	- Configure Clocks
	- On Partitions Page, choose: **Guided - use entire disk**
	- Choose a Mirror closest to you.
	- For Headless Installation, Only additionally check **SSH server**.
	- Choose to install GRUB on the **primary devices**.

Since, it is headless you don't have a gui. Connect through SSH.