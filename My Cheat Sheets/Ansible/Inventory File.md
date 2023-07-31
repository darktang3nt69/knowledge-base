1. Inventory file is where we defined the managed hosts for ansible to work/automate.
2. Inventory file is a simple INI file containing hosts, vars abouts the hosts etc.
3. Unless specified, ansible uses the `host` file located at `/etc/ansible/host`.
4. Sample Inventory file:
    ``` ini
    server1.company.com
    server2.company.com
    server3.company.com
    server4.company.com
    ```
5. Aliasing hosts. Here ansible_host is a predefined variable:
    ```ini
    #Below is the format
    alias ansible_host=<FQDN>
    web1 ansible_host=server1.company.com
    web2 ansible_host=server2.company.com
    web3 ansible_host=server3.company.com
    db1 ansible_host=server4.company.com
    ```
6. More Ansible variables:
     1. **ansible_host**: represents the FQDN/URL for the Managed Node.
     2. **ansible_connection**: States the connection to use to connect to the host. `ssh` for Linux and `winrm` for windows.
     3. **ansible_user**: Which user to use. For ex: `root` for Linux and 'administrator' for Windows.
     4. **ansible_password/ansible_ssh_pass**: `ansible_password` for Windows and   `ansible_ssh_pass` for linux
     ```ini
     #Web Servers
     web1 ansible_host=server1.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
     
     web2 ansible_host=server2.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
     
     web3 ansible_host=server3.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
     
     db1 ansible_host=server4.company.com ansible_connection=winrm ansible_user=administrator ansible_password=Dbp@aa123!>)
     ```
     
7. Grouping Hosts:
    ```ini
    #Web Servers
    web1 ansible_host=server1.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
    web2 ansible_host=server2.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
    web3 ansible_host=server3.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
    
    #Database Servers
    db1 ansible_host=server4.company.com ansible_connection=winrm ansible_user=administrator ansible_password=Password123!
    
    
    [web_servers]
    web1
    web2
    web3
    
    [db_servers]
    db1
    ```
8. Group of groups:
    ```ini
    [parent_group:children] child_group1 child_group2
    #For Example
    [all_servers:children]
    web_servers
    db_servers
    ```