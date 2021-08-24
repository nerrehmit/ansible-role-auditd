ansible-role-auditd
===================

Manage the auditd system, create rules and define its behaviour.
For an extensive guide on how to setup system auditing please refer to the
official [Red Hat System Auditing Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/chap-system_auditing).

A Short Auditd Rule Documentation
---------------------------------

The linux auditing daemon supports a wide variety of different rules. These can be grouped into the following rule classes.

## Control Rules

These rules allow to modify the audit system's behavior and some of its configs.

### Options

```bash
# set the max amount of audit buffer in the kernel
$ auditctl -b 8192
# set the action that is performed when an error is detected. 2=kernelpanic
$ auditctl -f 2
# enable/disable the audit system or lock the configuration. 2=lock
$ auditctl -e 2
# set the rate of generated messages per second. 0=nolimit
$ auditctl -r 0
# report the status of the audit system
$ auditctl -s
# list all currently loaded audit rules
$ auditctl -l
# delete all rules currently loaded by the audit system
$ auditctl -D
```

## System Call Rules

These rules allow the logging of system calls that any specified program generates.

### Options

```bash
$ auditctl -a action,filter -s system_call -F field=value -k comment
```

* `action` and `filter` specify when a certain event is logged.
  * `action` can be either `always` or `never`
  * `filter` can be one of the following: `task`, `exit`, `user`, `exclude`
* `system_call` specifies the system call by its name
  * see `/usr/include/asm/unistd_64.h` for a list of all possible syscalls.
* `field=value` specifies additional options that further modify the rule to match events based on a specified architecture, group, id, pid.
  * see `man 8 auditctl` for a complete list
* `comment` is an optional string that helps identify which rule or set of rules generated a particular log entry.

### Examples

```bash
$ auditctl -a always,exit -F arch=b64 -S adjtimex -S settimeofday -k time_change
```

- Create a log entry each time `adjtimex` or `settimeofday` sys calls are used by a program, on a 64bit architecture.


```bash
$ auditctl -a always,exit -S unlink -S rename -S renameat -F auid>=1000 -F auid!=4294967295 -k delete
```

- Defines a rule that creates a log entry every time a file is deleted or renamed by a user whose id is 1000 or larger.
- The option `-F auid!=4294967295` is used to exclude users whose login uid is not set.

## File System Rules

Also known as file watches, these rules allow the auditing of accesses to a file or directory from any program.

### Options

```bash
$ auditctl -w path_to_file -p permissions -k comment
```

* `path_to_file` is the file or directory that is audited
* `permissions` are the permissions that are logged
  * `r` - read acces to a file or directory
  * `w` - write acces to a file or directory
  * `x` - execute access to a file or directory
  * `a` - change in the file or directories attributes

### Example

```bash
$ auditctl -w /etc/passwd -p wa -k passwd_changes
```

- Create a rule that logs each write access or attribute change of /etc/passwd

## Executable File Rules

These rules allow the logging of executables.

### Options

```bash
$ auditctl -a always,exit -F exe=path_to_exe -k comment
```

* `action` and `filter` see above.
* `system_call` see above.
* `comment` see above.
* `path_to_exe` is the absolute path to the executable file.

### Example

```bash
$ auditctl -a always,exit -F exe=/bin/id -F arch=b64 -S execve -k execute_bin_id
```

- Create a rule that logs all executions of the /bin/id program.


Requirements
------------

None.

Role Variables
--------------

To define different types of system auditing rules use the following variable/syntax.

```yaml
auditd_custom_rules:
  # define a file system rule
  - type: filesystem
    file: /etc/passwd
    permissions: wa
    comment: passwd_changes
  # define a system call rule
  - type: syscall
    action: always,exit
    filters:
      - arch=b64
    syscalls:
      - adjtimex
      - settimeofday
    comment: time_change
  # define an executable rule
  - type: executable
    action: always,exit
    filters:
      - arch=b64
    executable: /bin/id
    comment: execution_bin_id
```

All the configurations for the audit daemon are configurable as variables. See `defaults/main.yaml` for more details.

Dependencies
------------

None.

Example Playbook
----------------

```yaml
---

- name: auditd test play
  hosts: all
  become: true
  vars:
  auditd_custom_rules:
    - type: filesystem
      file: /etc/passwd
      permissions: wa
      comment: passwd_changes
    - type: syscall
      action: always,exit
      filters:
        - arch=b64
      syscalls:
        - adjtimex
        - settimeofday
      comment: time_change
    - type: executable
      action: always,exit
      filters:
        - arch=b64
      executable: /bin/id
      comment: execution_bin_id
  roles:
    - auditd
```

License
-------

GPLv3

Author Information
------------------

Aaron Schmocker (aaron@0x29a.ch)
