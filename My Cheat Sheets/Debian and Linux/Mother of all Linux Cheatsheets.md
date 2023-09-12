1.  **SCP (Secure Copy)** :
    ```bash
    scp -r /path/to/local/folder username@remote_host:/path/to/destination/folder 
    ```
    
2. set environment variables in shell script,use `export` command 
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

6. Add User to Sudoers file : 
    ```bash
    echo "lokesh  ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/lokesh
    ```