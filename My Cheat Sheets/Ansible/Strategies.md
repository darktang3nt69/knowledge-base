### 1. Linear
- Default strategy in ansible. In this, Tasks are executed on managed hosts one after the other.
- It waits for all of the managed hosts to complete the current task before moving onto the next.
- By default, Ansible can communicate with 5 hosts at a time unless `forks` var is changed in `ansible.cfg`.
- You can select a different strategy for each play as shown above, or set your preferred strategy globally in `ansible.cfg`, under the `defaults` stanza.
    ```yaml
    [defaults]
    strategy = free
    ```

### 2. Serial or batch processing
- By default, Ansible runs in parallel against all the hosts in the `pattern` you set in the `hosts:` field of each play. If you want to manage only a few machines at a time, for example during a rolling update, you can define how many hosts Ansible should manage at a single time using the `serial` keyword:
    ```yaml
    - name: test play
      hosts: webservers
      #Handles first 2 and then next 3 and final 5 or remaining managed hosts. If only one value is given, then it only handles x no of managed hosts at a time.
      serial: 
          - 2
          - 3
          - 5
      gather_facts: False
      tasks:
        - name: first task
          command: hostname
        - name: second task
          command: hostname
    ```

### 3. Free
 - Task execution is as fast as possible per batch as defined by `serial` (default all). Ansible will not wait for other hosts to finish the current task before queuing more tasks for other hosts. All hosts are still attempted for the current task, but it prevents blocking new tasks for hosts that have already finished.
    
- With the free strategy, unlike the default linear strategy, a host that is slow or stuck on a specific task won’t hold up the rest of the hosts and tasks