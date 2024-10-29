1. Put OrangePi5 in Maskrom Mode. By pressing the bootloader button and then powering on the Pi.
2. Connect the pi to PC.
3. Launch a linux VM in virtual box.
4. Install `rkdeveloptool`.
    ```bash
    sudo apt update
    sudo apt install rkdeveloptool -y
    ```
5. Check if pi is detected:
    ```bash
    sudo rkdeveloptool id
    ```
6. Flash Bootloader:
    ```bash
    sudo rkdeveloptool db MiniLoaderAll.bin
    ```
7. Flash Loader:
    ```bash
    sudo rkdeveloptool ul MiniLoaderAll.bin
    ```
8. Write `rkspi_loader.img`:
    ```bash
    sudo rkdeveloptool wl 0 rkspi_loader.img
    ```
9. Test Device:
    ```bash
    sudo rkdeveloptool td
    ```
10. Reset Device:
    ```bash
    sudo rkdeveloptool rd
    ```
11. Insert NVME and enjoy.