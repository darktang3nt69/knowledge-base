### [Blocks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_blocks.html#blocks "Permalink to this heading")

1. Blocks create logical groups of tasks. Blocks also offer ways to handle task errors, similar to exception handling in many programming languages:
    - [Grouping tasks with blocks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_blocks.html#grouping-tasks-with-blocks)
    - [Handling errors with blocks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_blocks.html#handling-errors-with-blocks)

2. #### [Grouping tasks with blocks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_blocks.html#id6)
    1. All tasks in a block inherit directives applied at the block level. 
    2. Most of what you can apply to a single task (with the exception of loops) can be applied at the block level, so blocks make it much easier to set data or directives common to the tasks. 
    3. The directive does not affect the block itself, it is only inherited by the tasks enclosed by a block. For example, a when statement is applied to the tasks within a block, not to the block itself.
    ```yaml
   #Block example with named tasks inside the block
   tasks:
   - name: Install, configure, and start Apache
     block:       
      - name: Install httpd and memcached
        ansible.builtin.yum:
          name:
           - httpd
           - memcached
          state: present

      - name: Apply the foo config template
        ansible.builtin.template:
          src: templates/src.j2
          dest: /etc/foo.conf

      - name: Start service bar and enable it
        ansible.builtin.service:
         name: bar
         state: started
         enabled: True
     #the ‘when’ condition will be evaluated before Ansible runs each of the three tasks in the block.
     when: ansible_facts['distribution'] == 'CentOS'
     #All three tasks also inherit the privilege escalation directives, running as the root user. 
     become: true
     become_user: root
     #Finally, `ignore_errors: true` ensures that Ansible continues to execute the playbook even if some of the tasks fail.
     ignore_errors: true
    ```

3. #### Handling errors with blocks
    1. You can control how Ansible responds to task errors using blocks with `rescue` and `always` sections.
    2. Rescue blocks specify tasks to run when an earlier task in a block fails. This approach is similar to exception handling in many programming languages. 
    3. Ansible only runs rescue blocks after a task returns a ‘failed’ state. Bad task definitions and unreachable hosts will not trigger the rescue block.
    ```yaml   
       # 1. Rescue blocks specify tasks to run when an earlier task in a block fails.
       # 2. Ansible only runs rescue blocks after a task returns a ‘failed’ state.
       # 3. Bad task definitions and unreachable hosts will not trigger the rescue block.  name: learning blocks
       # 4. The rescued task is considered successful, and does not trigger `max_fail_percentage` or `any_errors_fatal` configurations. However, Ansible still reports a failure in the playbook statistics.
       # 5. You can use blocks with `flush_handlers` in a rescue task to ensure that all handlers run even if an error occurs
      - name: learning blocks
          hosts: all
          tasks:
            - name: Attempt and graceful roll back demo
              block:
                - name: Print a message
                  ansible.builtin.debug:
                    msg: 'I execute normally'
                  changed_when: true
                  notify: Run me even after an error
                  
                  # Goes to rescue section
                - name: Force a failure
                  ansible.builtin.command: /bin/false
                  
                - name: Never print this
                  ansible.builtin.debug:
                    msg: 'I never execute, due to the above task failing, :-('
                    
              # Only executed when task(s) fails. Like Catch block in python
              rescue:
                - name: Print when errors
                  ansible.builtin.debug:
                    msg: 'I caught an error'
                    
                - name: Force a failure in middle of recovery! >:-)
                  ansible.builtin.command: /bin/false
                  
                - name: Never print this
                  ansible.builtin.debug:
                    msg: 'I also never execute :-('
                    
              # Always executed. Like Finally block in python
              always:
                - name: Always do this
                  ansible.builtin.debug:
                    msg: "This always executes"
                    
                - name: Make sure all handlers run
                  meta: flush_handlers
                  
          handlers:
            - name: Run me even after an error
              ansible.builtin.debug:
                msg: 'This handler runs even on error'
    ```