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
| `systemctl start/restart/stop/enable foo.service`   | To start or restart or stop or enable a service                                                                                                                 |
| `systemctl --user start/restart/stop emacs.service` | To start or restart or stop per-user service                                                                                                                    |
| `systemctl list-units -at service`                  | To see all service units                                                                                                                                        |
| `systemctl list-dependencies foo.service`           | To list the dependencies of a service. when no service name is specified, lists the dependencies of default.target. add -all to expand dependencies recursively |
| `systemctl mask/unmask unit`                        | Mask/Unmask a unit to prevent enablement and manual activation                                                                                                  |
| `systemctl enable/disable --now foo.service`        | enable or disable a service right away                                                                                                                          |


#### 6. journalctl
| Command                                                | Description                                               |
| ------------------------------------------------------ | --------------------------------------------------------- |
| `sudo journalctl --unit=sshd.service -n 20 --no-pager` | Get logs of the unit=sshd.service, 20 lines with no pager |
| `sudo journalctl -p err`                               | Get error logs using priority.                            |
| `sudo journalctl -p info -g '^c'`                      | Get the `info` priority logs that begin with letter `c`   |
|                                                        |                                                           |

#### 7. ps
| Command  | Description                  |
| -------- | ---------------------------- |
| `ps u 1` | get process info about pid 1 |
|          |                              |

#### 8. df
| Command | Description                                    |
| ------- | ---------------------------------------------- |
| `df -h` | Disk free and show it in human readable format |
|         |                                                |

#### 9. du
| Command       | Description                                                               |
| ------------- | ------------------------------------------------------------------------- |
| `du -sh path` | Shows the disk space used by that dirs. s = summarize, h = human readable |
|               |                                                                           |
|               |                                                                           |

#### 10. chcon, setenforce
| Command                                             | Description                                                           |
| --------------------------------------------------- | --------------------------------------------------------------------- |
| `sudo chcon -t httpd_sys_content_t /var/index.html` | Change SElinux context to `httpd_sys_content_t`                       |
| `sudo setenforce 0`                                 | Temporarily change the SELinux status to `Permissive` on this system. |
| `sudo semanage user -l`                             | Get SElinux user Info.                                                                      |

#### 11. useradd, userdel (sudo):
| Command                                                                   | Description                                                                                                                                                                                    |
| ------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `sudo useradd new_user`                                                   | Creates a new user and group(default grp too) with the same name. 1. `/home/new_user` --> Home Directory. 2. `/bin/bash` --> Default Shell. 3. `/etc/skel` content will be copied to home dir. |
| `sudo useradd --defaults`                                                 | Shows user default settings                                                                                                                                                                    |
| `cat /etc/login.defs`                                                     | Other defaults regarding to account creation can be seen here. Comments are there for the defaults                                                                                             |
| `sudo userdel username`                                                   | user is deleted, user grp might get auto removed but not the home dir of the user                                                                                                              |
| `sudo userdel --remove username`                                          | Remove user along with the home dir.                                                                                                                                                           |
| `sudo useradd --shell /bin/othershell --home-dir /home/otherdir new_user` | Specify shell and home dir                                                                                                                                                                     |
| `sudo useradd --uid 6969 --groups new_user`                               | Create a user with specific uid and add the user to the supplied groups list.                                                                                                                                                                |
| `cat /etc/passwd`                                                         | Verbose info about all the users. Ex: `lokesh:x:1001:1002:Lokesh,,,:/home/lokesh:/usr/bin/zsh`                                                                                                 |
| `id`                                                                      | Get Verbose info about the current logged in user                                                                                                                                              |
| `sudo useradd --system username`                                          | Creates a System account which are used by daemons, programs etc. No home dir is created                                                                                                       |
|                                                                           |                                                                                                                                                                                                |

#### 12. usermod(sudo, only works for existing user):
| Command                                                   | Description                                                                                     |
| --------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `sudo usermod --home /home/otherdir --move-home username` | Copy contents of existing user's home dir to the new location and make it the default home dir. |
| `sudo usermod --login new_username old_username`          | Change username of existing user.                                                               |
| `sudo usermod --shell /bin/othershell username`           | Change default shell of existing user                                                           |
| `sudo usermod --lock username`                            | Locks user out. Can login using ssh if it was setup in the past.                                |
| `sudo usermod --expire YEAR-MONTH-DAY username`           | Set an account expiration date.                                                                 |
| `sudo usermod --expire "" username`                       | Never expire account. use `-1` to immediately expire the account                                |
| `sudo usermod -aG groupname username`                     | Add a user to a group                                                                           |
|                                                           |                                                                                                 |

#### 13. chage(Change Age):
| Command                                       | Description                                             |
| --------------------------------------------- | ------------------------------------------------------- |
| `sudo chage --list username`                  | List password information for the user                  |
| `sudo chage --maxdays 10 username`            | Enable password expiration in 10 days                   |
| `sudo chage --maxdays -1 username`            | Disable password expiration                             |
| `sudo chage --expiredate YYYY-MM-DD username` | Set account expiration date                             |
| `sudo chage --lastday 0 username`             | Force user to change password on next log in            |
| `sudo chage --warndays 2 jane`                | Warn `jane` atleast 2 days before to reset her password |
|                                               |                                                         |


#### 14. groupdel, groupmod, groupadd
| Command                                      | Description                                            |
| -------------------------------------------- | ------------------------------------------------------ |
| `sudo groupmod --new-name new_name old_name` | Rename a group                                         |
| `sudo groupdel group_name`                   | Deletes a group unless it is the primary grp of a user |
| `sudo groupadd -g 9875 grpname`              | Create a group with the specified `GID`.               |
|                                              |                                                        |


#### 15. nmcli, hostnamectl
| Command                                                                                                              | Description                            |
| -------------------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| `nmcli d wifi connect <ssid> password <pass> iface <wiface>`                                                         | Connect to a wireless access point     |
| `nmcli d wifi disconnect iface <wiface>`                                                                             | Disconnect from WIFI                   |
| `nmcli con`                                                                                                          | Show all available connections         |
| `nmcli radio wifi <on or off>`                                                                                       | Enable / Disable WIFI                  |
| `sudo nmcli connection modify <iface> autoconnect yes`                                                               | Auto connect to that iface.            |
| `sudo nmcli device reapply eth`                                                                                      | Reapply conn settings to that iface.   |
| `hostnamectl`                                                                                                        | Get the hostname of the computer       |
| `sudo hostnamectl set-hostname "hostname"`                                                                           | Set the hostname of the computer       |
| `sudo hostnamectl set-hostname --static "hostname.example.com" && sudo hostnamectl set-hostname --pretty "hostname"` | Set a pretty hostname for the computer |
| `sudo hostnamectl set-hostname --pretty ""`                                                                          | Reset hostname to its default value    |
|                                                                                                                      |                                        |

