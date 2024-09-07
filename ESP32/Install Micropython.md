1. Get firmware from : https://micropython.org/download/ESP32_GENERIC/
2. Install ESPtool python package:
    ```bash
    pip install esptool
    ```
3. Erase Flash, after executing this command Press and hold boot till the erasing starts:
    ```bash
    python -m esptool --chip esp32 --port COM3 erase_flash
    ```
4. Flash MicroPython, after executing this command Press and hold boot till the flashing starts:
    ```bash
    python -m esptool --chip esp32 --port COM3 --baud 460800 write_flash -z 0x1000 .\ESP32_GENERIC-20240222-v1.22.2.bin
    ```
