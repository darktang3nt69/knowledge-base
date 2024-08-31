In Ansible, "magic variables" are special variables that Ansible automatically populates and makes available during the execution of playbooks. These variables provide information about the execution environment, such as details about the host being managed, the playbook, or the task being executed. They are called "magic" because you don't need to define them—they are automatically available.

Here’s a list of some of the most commonly used magic variables in Ansible:

### 1. **Host and Group Variables**
- **`inventory_hostname`**: The name of the current host as it is defined in the inventory.
- **`inventory_hostname_short`**: The short hostname of the current host, without the domain part.
- **`group_names`**: A list of all the groups that the current host belongs to.
- **`groups`**: A dictionary containing all the groups in the inventory and the hosts within them.
- **`ansible_hostname`**: The actual hostname of the current host.

### 2. **Playbook and Task Variables**
- **`playbook_dir`**: The directory where the current playbook is located.
- **`playbook_path`**: The full path to the playbook that is being executed.
- **`role_path`**: The path to the current role being executed, available within a role's tasks, handlers, etc.
- **`ansible_playbook_python`**: The path to the Python interpreter used to execute the Ansible playbook.
- **`ansible_version`**: Information about the version of Ansible that is running.

### 3. **Execution Variables**
- **`ansible_facts`**: A dictionary of facts collected about the target host, such as IP addresses, OS version, and more. These are gathered by the `setup` module.
- **`hostvars`**: A dictionary containing the variables for all hosts in the inventory. You can use `hostvars['hostname']` to access variables for a specific host.
- **`ansible_verbosity`**: The verbosity level of the Ansible execution, ranging from 0 (normal) to 4 (max verbosity).
- **`ansible_check_mode`**: A boolean variable that is `True` if Ansible is running in check mode (`--check`), where no changes are actually made.
- **`ansible_diff_mode`**: A boolean variable that is `True` if Ansible is running in diff mode (`--diff`), showing the changes that would be made without actually applying them.

### 4. **Connection Variables**
- **`ansible_connection`**: The connection type Ansible is using to communicate with the host (e.g., `ssh`, `local`, `winrm`).
- **`ansible_user`**: The remote user Ansible is using to connect to the host.
- **`ansible_ssh_host`**: The hostname or IP address that Ansible is using for SSH connections.
- **`ansible_ssh_port`**: The port number Ansible is using for SSH connections.

### 5. **Loop Variables**
- **`item`**: When using loops, `item` holds the current item in the loop. If you're looping over a list of items, `item` will represent each item in turn.
- **`ansible_loop`**: Provides information about the loop's current index and other loop-related details.

### 6. **Special Variables**
- **`hostvars[inventory_hostname]`**: Accesses all variables for the current host.
- **`ansible_play_hosts`**: A list of all the hosts that are part of the current play.
- **`ansible_play_batch`**: The current set of hosts being worked on in parallel in the current play.
- **`omit`**: A special variable used to skip a parameter in a module if a condition is not met (e.g., if a variable is undefined).

These magic variables are essential for making playbooks dynamic and responsive to the environment in which they run. They allow you to adapt your tasks based on the target host's characteristics, the playbook context, and other factors.