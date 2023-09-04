# Lesson 14 Managing Processes and Tasks

- [Lesson 14 Managing Processes and Tasks](#lesson-14-managing-processes-and-tasks)
    - [14.1 Understanding Modules for Managing Processes and Tasks](#141-understanding-modules-for-managing-processes-and-tasks)
      - [Modules for Managing Processes and Tasks](#modules-for-managing-processes-and-tasks)
      - [Understanding the cron Module](#understanding-the-cron-module)
    - [14.2 Implementing a Playbook to Manage Processes and Tasks](#142-implementing-a-playbook-to-manage-processes-and-tasks)
    - [Lesson 14 Lab: Managing Processes and Tasks](#lesson-14-lab-managing-processes-and-tasks)
    - [Lesson 14 Lab Solution: Managing Processes and Tasks](#lesson-14-lab-solution-managing-processes-and-tasks)

### 14.1 Understanding Modules for Managing Processes and Tasks

#### Modules for Managing Processes and Tasks

- **cron**: Used cron to schedule a job.
- **at**: Uses at to run a turure job.
- **file**: Because some tasks simply don't have a specific Ansible module.
- **service**: Manages Services.
- **service_facts**: Uses information about services as facts.
- **systemd**: Manages Systemd Services.

#### Understanding the cron Module

- The **cron** module has a **name** option, which is useful to uniquely identify entries in crontab.
- Notice that this option has no meaning for Cron, but it helps Ansible in managing crontab entries.

### 14.2 Implementing a Playbook to Manage Processes and Tasks

```bash
# cron
cd lesson14/
vim setup-crontab.yml;
ansible-playbook setup-crontab.yml;
ansible ansible2.example.com -a "ls /etc/cron.d"
ansible ansible2.example.com -a "cat /etc/cron.d/keep-alive-messages"
vim delete-cron-job.yml
ansible-playbook delete-cron-job.yml;

# at
vim setup-at-task.yml
ansible-playbook setup-at-task.yml
ansible ansible2.example.com -a "which at"
ansible-doc yum;
ansible all -m yum -a "name=at state=present"
ansible ansible2.example.com -a "which at"
ansible-playbook setup-at-task.yml

# boot
vim setup-boot-target.yml
ansible-playbook -v setup-boot-target.yml;
```

### Lesson 14 Lab: Managing Processes and Tasks

- Set the default boot state of the file servers to multi-user.target.
- Reboot your server after doing so.
- Configure your playbook such that it will show the message "successfully rebooted" once it is available again.

### Lesson 14 Lab Solution: Managing Processes and Tasks

- vim lab.yml

```yaml
---
- name: set default boot target and reboot
  hosts: ansible2.example.com
  tasks:
  - name: set default boot target
    file:
      src: /usr/lib/systemd/system/multi-user.target
      dest: /etc/systemd/system/default.target
      state: link
  - name: reboot hosts
    reboot:
      msg: reboot initiated by Ansible
      test_command: whoami
  - name: print message to show host is back
    debug:
      msg: successfully rebooted
```
```bash
ansible-playbook lab.yml;
ansible ansible2.example.com -a "uptime"
```
