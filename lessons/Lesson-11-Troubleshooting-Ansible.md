# Lesson 11 Troubleshooting Ansible

- [Lesson 11 Troubleshooting Ansible](#lesson-11-troubleshooting-ansible)
    - [11.1 Managing Ansible Logs](#111-managing-ansible-logs)
      - [Understanding Ansible Logging](#understanding-ansible-logging)
    - [11.2 Using the Debug Module](#112-using-the-debug-module)
      - [Using the debug Module](#using-the-debug-module)
    - [11.3 Using Check Mode](#113-using-check-mode)
      - [Using Check Mode](#using-check-mode)
      - [Using Check Mode on Templates](#using-check-mode-on-templates)
    - [11.4 Using Modules for Troubleshooting and Testing](#114-using-modules-for-troubleshooting-and-testing)
      - [Understanding Modules to Check](#understanding-modules-to-check)
      - [Understanding stat](#understanding-stat)
    - [11.5 Troubleshooting Connection Issues](#115-troubleshooting-connection-issues)
      - [Understanding Connection Issues](#understanding-connection-issues)
      - [Analyzing Authentication Issues](#analyzing-authentication-issues)
      - [Connecting to Managed Hosts](#connecting-to-managed-hosts)
      - [Using Ad Hoc Commands to Test Connectivity](#using-ad-hoc-commands-to-test-connectivity)
    - [11.6 Analyzing Playbook Errors](#116-analyzing-playbook-errors)
      - [Analyzing Playbooks](#analyzing-playbooks)
    - [11.7 Avoiding Errors in Playbook Best Practices](#117-avoiding-errors-in-playbook-best-practices)
      - [Best Practice](#best-practice)
    - [Lesson 11 Lab: Troubleshooting Ansible](#lesson-11-lab-troubleshooting-ansible)
    - [Lesson 11 Lab Solution: Troubleshooting Ansible](#lesson-11-lab-solution-troubleshooting-ansible)

### 11.1 Managing Ansible Logs

#### Understanding Ansible Logging

- By default Ansible in is not configured to log its output anywhere.
- Set **log_path** in ansible.cfg to write logs to a specific destination.
  - Create this file in the project directory, /var/log is not writable by the Ansible user and will only work when runnign the playbook with sudo.
- When using this, you should also use log rotation.

```bash
vim ansible.cfg 

log_path = app.log # include it in defaults block.
ansible-playbook vsftpd.yml;
ansible-playbook --syntax-check vsftpd.yml;
vim vsftpd.yml; # Fix the syntax error.
ansible-playbook --syntax-check vsftpd.yml;
ansible-playbook vsftpd.yml;
```

### 11.2 Using the Debug Module

#### Using the debug Module

- Variables play an important role in playbooks.
- The **debug** module is used to show values of variables in playbooks.
- it can also be used to show messages in specific erro situations.

```bash
ansible-doc debug;
vim debugme.yml;
ansible-playbook debugme.yml;
ansible-playbook -v debugme.yml;
```

### 11.3 Using Check Mode

#### Using Check Mode

- Use **ansible-playbook --check** on a playbook to perform check mode; this will show what would happen when running the playbook without actually changing anything.
  - Moduels in the playbook mut support check mode.
  - Check mode doesn't always work well in conditionals.
- Set **check_mode: yes** within a task to always run that specific task in check mode.
  - This is useful for checking individual tasks.
  - When setting **check_mode: no** for a task, this task will never run in check mode.
  - Notice that **check_mode: no** is new since Ansible 2.6, and replaces the **always_run: yes** option earlier versions.

#### Using Check Mode on Templates

- Add **--diff** to an Ansible playbook run to see differences that would be made by template files on a managed hosts.
  - **ansible-playbook --check --diff myplaybook.yml**

```bash
ansible-playbook --check --diff vsftpd.yml
```

### 11.4 Using Modules for Troubleshooting and Testing

#### Understanding Modules to Check

- **uri**: checks content that is returned from a specific URL.
- **script**: runs a script from the control node on the manged hosts.
- **stat**: checks the status of files; use it to register a variable and next test to determine if a file exists.
- **assert**: this module will fail with an error if a specific condition is not met.

#### Understanding stat

- The **stat** module cab be used to check on file status.
- it returns a dictionary that contains a stat field which can have multiple values.
  - **atime**: last access time of the file.
  - **isdir**: true if file is a directory.
  - **exists**: true if file exists.
  - **size**: size in bytes.
  - and many more... 

```bash
vim bashversion.yml;
ansible-playbook bashversion.yml;
vim vim assertstat.yml;
ansible-playbook assertstat.yml;
```

### 11.5 Troubleshooting Connection Issues

#### Understanding Connection Issues

- Connection issues include the following.
  - Issues setting up the physical connectioin.
  - Issues running tasks as the target user.

#### Analyzing Authentication Issues

- Confirm the **remote_user** setting and existence of remote user on the managed host.
- Confirm host key setup.
- Verify **become** and **become_user**
- Check that **sudo** is configured correctly.

#### Connecting to Managed Hosts

- When a host is available at different IP addresses / names, you can use **ansible_host** in inventory to specify how to connect.
- this ensures that the connection is made in a persistent way, using the right interface.
- **web.example.com ansible_host=192.168.1.100**

#### Using Ad Hoc Commands to Test Connectivity

- The **ping** module was developed to test connectivity.
- Use the **--become** option to also test provilege escalation.
  - **ansible ansible1 -m ping**
  - **ansible ansible1 -m ping --become**
- Use the **command** module to test different items.
  - **ansible ansible1 -m command -a 'df'**
  - **ansible ansible1 -m command -a 'free -m'**
  - **ansible ansible1 -m shell -a 'df | grep sda'**

### 11.6 Analyzing Playbook Errors

#### Analyzing Playbooks

- Start by reading output messages.
- Next, add verbosity using **-v**
  - **-v**: the output data is displayed.
  - **-vv**: output as well as input data is shown.
  - - **-vvv**: adds connection information.
  - - **-vvvv**: adds additional information, for instance, about scripts that are executed and the user who's running tasks.

### 11.7 Avoiding Errors in Playbook Best Practices

#### Best Practice

- When developing Playbooks, some best practices should be applied.
- The **name** of plays and tasks should make sense to clearly see what's happening.
- Include comments to clarify difficult bits.
- Use white spaces to make playbooks more readable.
- Indentation is essential.
- Keep playbooks simple and small.
- Use includes when playbooks risk getting too big.

### Lesson 11 Lab: Troubleshooting Ansible

- Run the playbook in the lesson11/lab directory. Make sure to get out all errors.

### Lesson 11 Lab Solution: Troubleshooting Ansible

```bash
cd lesson11/lab/
ansible-playbook lab-copy-facts.yml;
ls;
vim ansible.cfg # change inventory = inventory to inventory = lab-inventory
ansible-playbook lab-copy-facts.yml;
ansible-playbook lab-playbook.yml;
ansible-playbook lab-copy-facts.yml -vvvv;
ansible-playbook lab-playbook.yml; # Fix the error.
ansible ansible1.example.com -m command -a "firewall-cmd --get-services"
ansible-playbook lab-playbook.yml;
```
