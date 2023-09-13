| Command | Description                                                                                                                                                        |                           |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------- |
| `apropos` | _apropos_ command helps the user when they don’t remember the exact command but knows a few keywords related to the command that define its uses or functionality. | apropos link              |
| `ln`      | Create hard links by default, symbolic links with --symbolic.                                                                                                      | ln -s /tmp ~/link_to_temp |


#### 1. ls:

| Command          | Description             |
| ---------------- | ----------------------- |
| `ls --full-time` | Show time in ISO format |


#### 2. find:
| Command                                        | Description                                                          |
| ---------------------------------------------- | -------------------------------------------------------------------- |
| `find /dev/ -mmin -5`                          | Find files which were modified within last 5 mins.                   |
| `find /dev/ -mmin +5`                          | Find files which were modified before the last 5 mins.               |
| `sudo find /var/log/ -perm -g=w ! -perm /o=rw` | the `group` can `write` to, but `others cannot read or write` to it. |
| `find /home/bob -size 213k -o -perm 402`       | Find files with exactly 213 kilobytes or with permission 402         |
|                                                |                                                                      |

#### 3. chmod:
| Command                              | Description                                                                       |
| ------------------------------------ | --------------------------------------------------------------------------------- |
| `chmod g-w file`                     | Removes the write access for the group.                                           |
| `chmod a+rx path/to/file `           | Give [a]ll users rights to [r]ead and e[x]ecute                                   |
| `chmod o=g path/to/file`             | Give [o]thers (not in the file owner's group) the same rights as the [g]roup      |
| `chmod o= path/to/file`              | Remove all rights from [o]thers                                                   |
| `chmod -R g+w,o+w path/to/directory` | Change permissions recursively giving [g]roup and [o]thers the ability to [w]rite |
| `chmod `                                     |                                                                                   |


#### 4. shutdown
| Command              | Description                           |
| -------------------- | ------------------------------------- |
| `sudo shutdown +120` | Schedule a shutdown after 120 minutes |
| `sudo shutdown -c`   | Cancels the shutdown                  |
|                      |                                       |

#### 5. systemctl
| Command                                             | Description                                                                                                                                                     |
| --------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `systemctl [VERB] [SERVICE]`                        | Start, stop, or restart a given service(s).                                                                                                                     |
| `systemctl is-active ufw`                           | Check if a given service(s) is active or failed. If it is, 'active' will display else, 'failed' will display.                                                   |
| `systemctl start/restart/stop/enable foo.service`   | To start or restart or stop or enable a service                                                                                                                          |
| `systemctl --user start/restart/stop emacs.service` | To start or restart or stop per-user service                                                                                                                          |
| `systemctl list-units -at service`                  | To see all service units                                                                                                                                        |
| `systemctl list-dependencies foo.service`           | To list the dependencies of a service. when no service name is specified, lists the dependencies of default.target. add -all to expand dependencies recursively |
| `systemctl mask/unmask unit`                        | Mask/Unmask a unit to prevent enablement and manual activation                                                                                                  |
| `systemctl enable/disable --now foo.service`        | enable or disable a service right away                                                                                                                          |