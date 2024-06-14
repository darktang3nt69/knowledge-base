1. **Ansible.cfg**: Depending the context, ansible chooses an `ansible.cfg` file. There are 4 locations that ansible will look for `ansible.cfg`  there is a preference to the said selection.       Below is a list of such location listed higher to lower preference:
    1. `ANSIBLE_CONFIG`: This env var overrides everything.
    2. `/etc/ansible/ansible.cfg`: This is the default location that ansible will check.
    3. `home directory of a user:`: exactly.
    4. `within current directory`: Looks for it in the current directory else, it searches in the home directory of the user.
2. **Jinja2**:
    ```yaml
    #No need for qoutes when specified in the same line.
    - name: example
      service: name{{ name }} state=present enabled=true
    #Need qoutes when specified as a list.
    - name: example
      service: 
        name: '{{ name }}'
        state: present
        enabled: true
    ```
3. **Async tasks**: Sometimes we do not want ansible to hold ssh connection till a command is completed or if it is taking longer than ssh timeout. We use:
    1. **async**: We tell ansible that this is an async task. Run it and check on it later.
    2. **poll**: we tell ansible how often we'd like to check status of the task. **Default value is 10 secs.**
    ```yaml
    - name: Monitor Web Application for 6 Minutes
      hosts: web_server
      command: /opt/monitor_webapp.py
      #Tell ansible to wait for 360 secs for the task to complete.
      async: 360
      #Tell ansible to check status every x secs. Here, it is 0 secs, meaning ansible doesn't wait for the task to finish. Moves onto the next task
      poll: 0
      register: webapp_result
    
    - name: Monitor Database for 6 Minutes
      hosts: db_server
      command: /opt/monitor_database.py
      async: 360
      poll: 0
      register: database_result

   - name: Check status of tasks
      #aync_status is used to monitor async tasks
      aync_status: jid={{ webapp_result.ansible_job_id }}
      register: job_result
      until: job_result.finished
      retries: 30
    ```