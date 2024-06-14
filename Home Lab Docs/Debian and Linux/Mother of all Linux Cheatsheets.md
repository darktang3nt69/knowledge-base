1.  **SCP (Secure Copy)** :
    ```bash
    scp -r /path/to/local/folder username@remote_host:/path/to/destination/folder 
    ```
    
2. Set environment variables in shell script, use `export` command 
    ```bash
    export MY_VAR="Hello, World"
    ```


3. Get cheat sheet for a command 
    ```bash
    curl https://cheat.sh/curl
    ```

4. Install a .deb File : 
    ```bash
    sudo dpkg -i path/to/file
    ```

5. Add User to Sudoers file : 
    ```bash
    echo "lokesh  ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/lokesh
    ```

6 . Change from `text-only` console mode to `graphical desktop`:
```bash
sudo systemctl set-default graphical.target
```

7. Change Default timeout for grub:
    ```bash
    sudo vi /etc/default/grub
    # Change GRUB_TIMEOUT=1.
    ```

8. Get details about the current kernel:
    ```bash
    cat /proc/version
    ```

9. **[Niceness](https://www.tecmint.com/set-linux-process-priority-using-nice-and-renice-commands/)**: Niceness in Linux means how 'nice' it is to other processes.  Process Priority in user space. Negative values can only be set by the root user.

10. **Process State Management:**
    1. Use `ctrl + c` -----> sends `SIGINT` signal to immediately **quit** the process.
    2. Use `ctrl + z` -----> sends `SIGTSTP` signal to immediately **pause** the process. It is entirely frozen, no progress is made.
    3. Return to the **paused app** using `fg` which is to bring the paused to **foreground**.
    4. To make a command run in the **background** use `&`. Example: 
        ```bash
        sleep 3000000000000000000 &
        [1] 2151131 # [1] is the job number. which can later be used to
                    # bring the background app to foreground
        ```
    5. Use `jobs` command to see a list of commands running **bg**:
        ```bash
        ❯ jobs
        [1]  + running    sleep 3000000000000000000
        ```
    6. use `fg` command to bring a `bg` task to foreground:
        ```bash
        fg 1 # Replace with job number
        ```
11. Set user limits:
    ```bash
    sudo nano /etc/security/limits.conf
    ```
12. Manage sudo rights:
    ```bash
    sudo visudo
    ```
13. Restrict root access to SSH service via `PAM`:
    ```bash
    sudo vi /etc/pam.d/sshd

    # Add this  line at the EOF of `/etc/pam.d/sshd`
    auth    required       pam_listfile.so onerr=succeed  item=user  sense=deny  file=/etc/ssh/deniedusers
    
    sudo vi /etc/ssh/deniedusers # Add `root` to this file.
    ```
14. Apply DNS globally edit:
    ```bash
    /etc/systemd/resolved.conf # edit this file

    sudo systemctl restart systemd-resolved.service # restart the daemon
    ```
15. Know your default gateway:
    ```bash
    ip route # Look for default route.
    ```
16. Find out what `process` is listening for incoming connections on port `22`, and identify its `PID`: `sudo ss -tlnup | grep :22`
17. Set Timezone:
    ```bash
    sudo timedatectl set-timezone Asia/Kolkata
    ```
18. Using `timedatectl` configure RTC (real-time clock) to maintain the RTC in local time.  RTC is a battery-powered computer clock that keeps track of the time even when the system is turned off.
    ```bash
    sudo timedatectl set-local-rtc 1
    ```
19. 