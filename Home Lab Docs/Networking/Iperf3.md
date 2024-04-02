### On Windows Machine
1. Download this binary: [Iperf3](https://iperf.fr/download/windows/iperf-3.1.3-win32.zip)
2. Open a command prompt in the location of iperf binary:
3. 
    1. -s: acts as a server.
    2. -p: run on a specific port.
    ```powershell
    .\iperf3.exe -s -p 7575
    ```

### On Linux Machine
1. Run the below command:
    1. IP of the server.
    ```
    iPerf3 -c <ip-of-server> -p 7575
    ```