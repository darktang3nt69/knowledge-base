1. Inventory file is where we defined the managed hosts for ansible to work/automate.
2. Inventory file is a simple INI file containing hosts, vars abouts the hosts etc.
3. Unless specified, ansible uses the `host` file located at `/etc/ansible/host`.
4. **Sample Inventory file**:
    ``` ini
    server1.company.com
    server2.company.com
    server3.company.com
    server4.company.com
    ```
5. **Aliasing hosts**. Here ansible_host is a predefined variable:
    ```ini
    #Below is the format
    alias_name ansible_host=<FQDN>
    web1 ansible_host=server1.company.com
    web2 ansible_host=server2.company.com
    web3 ansible_host=server3.company.com
    db1 ansible_host=server4.company.com
    ```
6. **More Ansible variables**:
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
     
7. **Grouping Hosts**:  Ansible creates two default groups: `all` and `ungrouped`. Every host will always belong to at least 2 groups (`all` and `ungrouped` or `all` and some other group).:
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
8. **Group of groups**: Child groups have a couple of properties to note:

    - Any host that is member of a child group is automatically a member of the parent group.
        
    - Groups can have multiple parents and children, but not circular relationships.
        
    - Hosts can also be in multiple groups, but there will only be **one** instance of a host at runtime. Ansible merges the data from the multiple groups.
    ```ini
    [parent_group:children] child_group1 child_group2
    #For Example
    [all_servers:children]
    web_servers
    db_servers
    ```
9. **Adding ranges of hosts**: If you have a lot of hosts with a similar pattern, you can add them as a range rather than listing each hostname separately:

    ```ini
    [webservers]
    www[01:50].example.com
    ---
    #includes increment
    [webservers]
    www[01:50:2].example.com: 
    ```
10. **Passing multiple inventory files**. This can be useful when you want to target normally separate environments, like staging and production, at the same time for a specific action:
    ```bash
    ansible-playbook get_logs.yml -i staging -i production
    ```
11. **Managing inventory load order**: Ansible loads inventory sources in ASCII order according to the filenames. If you define parent groups in one file or directory and child groups in other files or directories, the files that define the child groups must be loaded first. If the parent groups are loaded first, you will see the error `Unable to parse /path/to/source_of_parent_groups as an inventory source`.
 -    For example, if you have a file called `groups-of-groups` that defines a `production` group with child groups defined in a file called `on-prem`, Ansible cannot parse the `production` group. To avoid this problem, you can control the load order by adding prefixes to the files:
        ```bash
        inventory/
          01-openstack.yml # configure inventory plugin to get hosts from OpenStack cloud
          02-dynamic-inventory.py   # add additional hosts with dynamic inventory script
          03-on-prem                # add static hosts and groups
          04-groups-of-groups       # add parent groups
        ```
12. Assigning variable to a managed node:
    ```ini
    [atlanta]
    host1 http_port=80 maxRequestsPerChild=808
    host2 http_port=303 maxRequestsPerChild=909
    #Non standard ssh ports can be included like this
    badwolf.example.com:5309
    #To use localhost set ansible_connection to local:
    localhost              ansible_connection=local
    #Include username:
    other1.example.com     ansible_connection=ssh        ansible_user=myuser
    other2.example.com     ansible_connection=ssh        ansible_user=myotheruser
    ```
13. **Assigning a variable to many machines: group variables**: If all hosts in a group share a variable value, you can apply that variable to an entire group at once.
- Before executing, however, Ansible always flattens variables, including inventory variables, to the host level. If a host is a member of multiple groups, Ansible reads variable values from all of those groups. If you assign different values to the same variable in different groups, Ansible chooses which value to use based on internal [rules for merging](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#how-we-merge).
    ```ini
    [atlanta]
    host1
    host2
    
    [atlanta:vars]
    ntp_server=ntp.atlanta.example.com
    proxy=proxy.atlanta.example.com
    ```
14. **Inheriting variable values: group variables for groups of groups**:You can apply variables to parent groups (nested groups or groups of groups) as well as to child groups. A child group’s variables will have higher precedence (override) a parent group’s variables.
    ```bash
    [atlanta]
    host1
    host2
    
    [raleigh]
    host2
    host3
    
    [southeast:children]
    atlanta
    raleigh
    
    [southeast:vars]
    some_server=foo.southeast.example.com
    halon_system_timeout=30
    self_destruct_countdown=60
    escape_pods=2
    
    [usa:children]
    southeast
    northeast
    southwest
    northwest
    ```
