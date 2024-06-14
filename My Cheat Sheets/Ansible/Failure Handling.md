1. **Default Behavior**: Ansible tries to complete the playbook on as many servers as possible. When ever a host fails execution of a task, Ansible removes that host from the list and continues execution of playbook with the remaining hosts.

2.  **any_errors_fatal**: When this is enabled, if any of the task fail on one managed host, Ansible doesn't continue with the remaining hosts and exists the playbook unlike the default behavior.
    ```yaml
    - host: all
    name: Any errors fatal
    any_errors_fatal: true
    tasks:
      - task1
      - task2
      - task3
    ```
3. **Ignore_errors**: Continue with next tasks if one fails.
4. **failed_when**: Fail a task on a condition.
    ```yaml
    - name: example
      hosts: example
      tasks:
        - mail:
            to: devops@corp.com
            subject: Server Deployed
            body: Web Server Deployed
          #Continues with the next task even if this fails.
          ignore_errors: yes

        - command: cat /var/log/server.log
          register: command_output
          #Task is failed when the below condition is met.
          failed_when: "'ERROR' in command_output.stdout"
        
        - name: Fail task when the command error output prints FAILED
          ansible.builtin.command: /usr/bin/example-command -x -y -z
          register: command_result
          failed_when: "'FAILED' in command_result.stderr"
        
        - name: Fail task when both files are identical
          ansible.builtin.raw: diff foo/file1 bar/file2
          register: diff_cmd
          failed_when: diff_cmd.rc == 0 or diff_cmd.rc >= 2

        - name: Check if a file exists in temp and fail task if it does
          ansible.builtin.command: ls /tmp/this_should_not_be_here
          register: result
          #You can also combine multiple conditions for failure. 
          #This task will fail if both conditions are true
          failed_when:
            - result.rc == 0
            - '"No such" not in result.stdout'
    ```
5. **unreachable**: You can ignore a task failure due to the host instance being ‘UNREACHABLE’ with the `ignore_unreachable` keyword. Ansible ignores the task errors, but continues to execute future tasks against the unreachable host. For example, at the task level:
    ```yaml
    - name: This executes, fails, and the failure is ignored
  ansible.builtin.command: /bin/true
  ignore_unreachable: true

    - name: This executes, fails, and ends the play for this host
  ansible.builtin.command: /bin/true
  
  ---
  #At playbook level
  
    - hosts: all
      ignore_unreachable: true
      tasks:
       - name: This executes, fails, and the failure is ignored
        ansible.builtin.command: /bin/true
    
       - name: This executes, fails, and ends the play for this host
        ansible.builtin.command: /bin/true
        ignore_unreachable: false
    ```
6. **Handlers and Failures**: 
    1. Ansible runs [handlers](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_handlers.html#handlers) at the end of each play. 
    2. If a task notifies a handler but another task fails later in the play, by default the handler does _not_ run on that host, which may leave the host in an unexpected state. 
    3. For example, a task could update a configuration file and notify a handler to restart some service. If a task later in the same play fails, the configuration file might be changed but the service will not be restarted.
    4. You can change this behavior with the `--force-handlers` command-line option, by including `force_handlers: True` in a play, or by adding `force_handlers = True` to ansible.cfg. 
    5. When handlers are forced, Ansible will run all notified handlers on all hosts, even hosts with failed tasks. (Note that certain errors could still prevent the handler from running, such as a host becoming unreachable.)
7. **Defining 'Changed'**: 
    1. Ansible lets you define when a particular task has “changed” a remote node using the `changed_when` conditional. 
    2. This lets you determine, based on return codes or output, whether a change should be reported in Ansible statistics and whether a handler should be triggered or not.
    ```yaml
    tasks:
      - name: Report 'changed' when the return code is not equal to 2
        ansible.builtin.shell: /usr/bin/taylorswift --mode="blank space"
        register: bass_result
        changed_when: "bass_result.rc != 2"

      - name: This will never report 'changed' status
        ansible.builtin.shell: wall 'beep'
        changed_when: False

      - name: Combine multiple conditions to override 'changed' result
        ansible.builtin.command: /bin/fake_command
        register: result
        ignore_errors: True
        changed_when:
         - '"ERROR" in result.stderr'
         - result.rc == 2
    ```
8. **Ensuring Success and Failure for shell and command**: The [command](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/command_module.html#command-module) and [shell](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/shell_module.html#shell-module) modules care about return codes, so if you have a command whose successful exit code is not zero, you can do this:
    ```yaml
    tasks:
      - name: Run this command and ignore the result
        ansible.builtin.shell: /usr/bin/somecommand || /bin/true
    ```