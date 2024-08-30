[**Intro**](obsidian://open?vault=knowledge-base&file=My%20Cheat%20Sheets%2FAnsible%2FIntro%20to%20Ansible)

1. **Ansible.cfg**: Depending the context, ansible chooses an `ansible.cfg` file. There are 4 locations that ansible will look for `ansible.cfg` there is a preference to the said selection. Below is a list of such location listed higher to lower preference:
    
    1. `ANSIBLE_CONFIG`: This env var overrides everything.
    2. `/etc/ansible/ansible.cfg`: This is the default location that ansible will check.
    3. `home directory of a user:`: exactly.
    4. `within current directory`: Looks for it in the current directory else, it searches in the home directory of the user.
    **Remember that not all configs have to al values. They just need to override the specific value.**
```bash
ANSIBLE_CONFIG=/path/to/custom/config.cfg ansible-playbook playbook.yml # custom config just for this command/playbook. 
# sets temp var
```
2. **ENV**: To override any variable using env vars. set the env var in this format: `ANSIBLE_VAR_IN_CAPS=value`. all other values are overridden by the env var.
3. **Ansible config help**:
    1. `ansible-config list # Lists all configurations`
    2. `ansible-config view # shows current file`
    3. `ansible-config dump # shows active config after the precedence order. Also, shows where/how it picked that value.`
4. [**Inventory File**](obsidian://open?vault=knowledge-base&file=My%20Cheat%20Sheets%2FAnsible%2FInventory%20File)