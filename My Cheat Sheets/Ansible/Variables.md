[Variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html) in Ansible can defined in three places:
1. Local variables to the task. When you define variables in a play, they are only visible to tasks executed in that play:
    ```yaml
    - hosts: local
    name: 'Use local variables'
    vars:
        #Variable local to the task
        car_model: 'Alto 800'
        country_name: India
        title: 'Systems Engineer'
        tasks:                     #Uses jinja2 templating - '{{ variable }}'
        - command: 'echo "My car is {{ car_model }}"'
        - command: 'echo "I live in the {{ country_name }}"'
        - command: 'echo "I work as a {{ title }}"'
    ```
2. Can be set as host variables:
    ```ini
    local example_variable=example
    ```
3. List variables:
    ```yaml
    #Defining vars
    region:
      - northeast
      - southeast
      - midwest
    #Referencing vars
    region: "{{ region[0] }}"
    ```
4. Dictionary variables:
    ```yaml
    #Defining vars
    foo:
      field1: one
      field2: two
    #Referencing vars
    foo['field1'] #Or
    foo.field1
    ```
5. Defining vars at runtime:
    ```bash
    ansible-playbook release.yml --extra-vars "version=1.23.45 other_variable=foo"
    ```
6. Variables defined in external files, those files have to be imported:
    ```yaml
    - hosts: all
      remote_user: root
      vars:
        favcolor: blue
      vars_files:
        - /vars/external_vars.yml
      tasks:
        - name: This is just a placeholder
        ansible.builtin.command: /bin/echo foo
    ```
7. [Order of Preference](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence)