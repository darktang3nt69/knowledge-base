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