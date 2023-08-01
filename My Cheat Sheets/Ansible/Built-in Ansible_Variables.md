### Host connection:

1. **ansible_connection**: Connection type to the host. This can be the name of any of ansible’s connection plugins. SSH protocol types are `smart`, `ssh` or `paramiko`. The default is **smart**. Non-SSH based types are described in the next section.

#### General for all connections:

2. **ansible_host**: The name of the host to connect to, if different from the alias you wish to give to it.

3. **ansible_port**: The connection port number, if not the default (22 for ssh)

4. **ansible_user**: The user name to use when connecting to the host

5. **ansible_password**: The password to use to authenticate to the host (never store this variable in plain text; always use a vault. See [Keep vaulted variables safely visible](https://docs.ansible.com/ansible/latest/tips_tricks/ansible_tips_tricks.html#tip-for-variables-and-vaults))

####  Specific to the SSH connection:

6. **ansible_ssh_private_key_file**: Private key file used by ssh. Useful if using multiple keys and you don’t want to use SSH agent.

7. **ansible_ssh_common_args**: This setting is always appended to the default command line for **sftp**, **scp**, and **ssh**. Useful to configure a `ProxyCommand` for a certain host (or group).

8. **ansible_sftp_extra_args**: This setting is always appended to the default **sftp** command line.

9. **ansible_scp_extra_args**: This setting is always appended to the default **scp** command line.

10. **ansible_ssh_extra_args**: This setting is always appended to the default **ssh** command line.

11. **ansible_ssh_pipelining**: Determines whether or not to use SSH pipelining. This can override the `pipelining` setting in `ansible.cfg`.

12. **ansible_ssh_executable** (added in version 2.2): This setting overrides the default behavior to use the system **ssh**. This can override the `ssh_executable` setting in `ansible.cfg`.

#### Privilege escalation (see [Ansible Privilege Escalation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_privilege_escalation.html#become) for further details):

13. **ansible_become**: Equivalent to `ansible_sudo` or `ansible_su`, allows to force privilege escalation

14. **ansible_become_method**: Allows to set privilege escalation method

15. **ansible_become_user**: Equivalent to `ansible_sudo_user` or `ansible_su_user`, allows to set the user you become through privilege escalation

16. **ansible_become_password**: Equivalent to `ansible_sudo_password` or `ansible_su_password`, allows you to set the privilege escalation password (never store this variable in plain text; always use a vault. See [Keep vaulted variables safely visible](https://docs.ansible.com/ansible/latest/tips_tricks/ansible_tips_tricks.html#tip-for-variables-and-vaults))

17. **ansible_become_exe**: Equivalent to `ansible_sudo_exe` or `ansible_su_exe`, allows you to set the executable for the escalation method selected

18. **ansible_become_flags**: Equivalent to `ansible_sudo_flags` or `ansible_su_flags`, allows you to set the flags passed to the selected escalation method. This can be also set globally in `ansible.cfg` in the `sudo_flags` option

#### Remote host environment parameters:

19. **ansible_shell_type**: The shell type of the target system. You should not use this setting unless you have set the [ansible_shell_executable](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#ansible-shell-executable) to a non-Bourne (sh) compatible shell. By default commands are formatted using `sh`-style syntax. Setting this to `csh` or `fish` will cause commands executed on target systems to follow those shell’s syntax instead.

20. **ansible_python_interpreter**: The target host python path. This is useful for systems with more than one Python or not located at **/usr/bin/python** such as *BSD, or where **/usr/bin/python** is not a 2.X series Python. We do not use the **/usr/bin/env** mechanism as that requires the remote user’s path to be set right and also assumes the **python** executable is named python, where the executable might be named something like **python2.6**.

21. **ansible_*_interpreter**: Works for anything such as ruby or perl and works just like [ansible_python_interpreter](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#ansible-python-interpreter). This replaces shebang of modules which will run on that host.

**New in version 2.1.**

22. **ansible_shell_executable**: This sets the shell the ansible controller will use on the target machine, overrides `executable` in `ansible.cfg` which defaults to **/bin/sh**. You should really only change it if is not possible to use **/bin/sh** (in other words, if **/bin/sh** is not installed on the target machine or cannot be run from sudo.).

    Examples from an Ansible-INI host file:
    ```ini
    some_host         ansible_port=2222     ansible_user=manager
    aws_host          ansible_ssh_private_key_file=/home/example/.ssh/aws.pem
    freebsd_host      ansible_python_interpreter=/usr/local/bin/python
    ruby_module_host  ansible_ruby_interpreter=/usr/bin/ruby.1.9.3
    ```


#### [Non-SSH connection types](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#id22)

As stated in the previous section, Ansible executes playbooks over SSH but it is not limited to this connection type. With the host specific parameter `ansible_connection=<connector>`, the connection type can be changed. The following non-SSH based connectors are available:

23. **local**: This connector can be used to deploy the playbook to the control machine itself.

24. **docker**: This connector deploys the playbook directly into Docker containers using the local Docker client. The following parameters are processed by this connector:

  -  **ansible_host**
    
    The name of the Docker container to connect to.
    
  -  **ansible_user**
    
    The user name to operate within the container. The user must exist inside the container.
    
  -  **ansible_become**
    
    If set to `true` the `become_user` will be used to operate within the container.
    
    **ansible_docker_extra_args**
    
    Could be a string with any additional arguments understood by Docker, which are not command specific. This parameter is mainly used to configure a remote Docker daemon to use.

Here is an example of how to instantly deploy to created containers:
    ```yaml
    - name: Create a jenkins container
      community.general.docker_container:
        docker_host: myserver.net:4243
        name: my_jenkins
        image: jenkins
    
    - name: Add the container to inventory
      ansible.builtin.add_host:
        name: my_jenkins
        ansible_connection: docker
        ansible_docker_extra_args: "--tlsverify --tlscacert=/path/to/ca.pem --tlscert=/path/to/client-cert.pem --tlskey=/path/to/client-key.pem -H=tcp://myserver.net:4243"
        ansible_user: jenkins
      changed_when: false
    
    - name: Create a directory for ssh keys
      delegate_to: my_jenkins
      ansible.builtin.file:
        path: "/var/jenkins_home/.ssh/jupiter"
        state: directory
    ```
