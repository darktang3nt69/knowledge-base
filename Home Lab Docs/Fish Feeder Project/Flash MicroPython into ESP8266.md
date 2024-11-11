1. Download MicroPython bin file from [micropython](https://micropython.org/download/ESP8266_GENERIC/). and refer this article for more [info](https://www.embedded-robotics.com/esp8266-micropython/).
2. Make sure to have python installed and also install esptool:
    ```bash
    pip install esptool
    ```
3. Execute the following command to see if ESP3266 is detected:
    ```bash
    esptool chip_id
    ```
4. Erase old firmware:
    ```bash
    esptool --port COM3 erase_flash
    ```
5. Flash MicroPython ESP8266:
    ```bash
    # <serial_port> = COM Port in Windows (e.g., COM5) or /dev/ttyUSB* port in Linux (e.g., /dev/ttyUSB0)
    esptool --port COM3 write_flash --flash_size=detect -fm dio 0x00000 ESP8266_GENERIC-20240222-v1.24.2.bin
    ```
6. Flash ESP32:
```bash
esptool --chip esp32 --port COM3 write_flash -z 0x1000 ESP32_GENERIC-20241025-v1.24.0.bin
```