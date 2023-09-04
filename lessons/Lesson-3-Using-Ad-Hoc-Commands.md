# Lesson 3 Using Ad Hoc Commands

- [Lesson 3 Using Ad Hoc Commands](#lesson-3-using-ad-hoc-commands)
    - [3.1 Using Ad Hoc Commands](#31-using-ad-hoc-commands)
      - [Understanding Ad Hoc Commands](#understanding-ad-hoc-commands)
      - [Ad Hoc Commands Ingredients](#ad-hoc-commands-ingredients)
    - [3.2 Understanding Ansible Modules](#32-understanding-ansible-modules)
      - [Understanding Ansible Modules](#understanding-ansible-modules)
    - [3.3 Using ansible-doc to get Module Documentation](#33-using-ansible-doc-to-get-module-documentation)
      - [Using ansible-doc](#using-ansible-doc)
      - [Understanding Modules Status](#understanding-modules-status)
      - [Understanding Module Support Status](#understanding-module-support-status)
    - [3.4 Introducing Essential Ansible Modules](#34-introducing-essential-ansible-modules)
      - [Essential Ansible Module](#essential-ansible-module)
    - [Lesson 3 Lab: Using Ad Hoc Commands](#lesson-3-lab-using-ad-hoc-commands)
    - [Lesson 3 Lab Solution: Using Ad Hoc Commands](#lesson-3-lab-solution-using-ad-hoc-commands)

### 3.1 Using Ad Hoc Commands

#### Understanding Ad Hoc Commands

- To run tasks on Ansible, you will typically want to use Playbooks.
- A Playbook is a YAML file in which tasks can be defined.
- For some tasks, writing a playbook is just too much work.
- In such cases, you can use Ad hoc commands.
- Ad hoc commands are also useful for testing if your playbook was successfull.

#### Ad Hoc Commands Ingredients

- The basic structure is **ansible hosts -m module [-a 'module arguments'] [-i inventory]**
- The hosts part specifies on which host the command should be performed.
- The module part indicates which Ansible module to use, following the module you will specify module arguments.
- If inventory is not specified, you will need to specify the inventory as an argument.
- An example of an ad hoc command is **ansible all -m user -a "name=anil"**
- And to verify the command works, use **ansible all -m command -a "id anil"**
 
### 3.2 Understanding Ansible Modules

#### Understanding Ansible Modules

- Ansible comes with lots of modules that allow you to perform specific tasks on managed hosts.
- When using Ansible, you will always use modules to tell Ansible what you want it to do, in ad-hoc commands as well as in playbooks.
- many modules are provided with Ansible, if required you can develop your own modules.
- Use **ansible-doc -l** for a list of modules currently available.
- All modules work with arguments, **ansible-doc** will show which arguments are available and which are requiured.
- Ansible modules are idempotent: which means that when running them again they will give the same result and if a task has already been configure, they won't do it again.

```bash
ansible all -m user -a "name=anil state=absent"
ansible all -m command -a "id anil"
ansible all -m command -a "id ansible"
ansible all -m ping
```

### 3.3 Using ansible-doc to get Module Documentation

#### Using ansible-doc

- The **ansible-doc** command provides authritative Documentation about modules.
- Use **ansible-doc -l** for a list of all modules.
- Use **ansible-doc modulename** to get the Documentation for a specific module.

```bash
ansible-doc -l;
ansible-doc -l | wc;
ansible-doc user;
```

#### Understanding Modules Status

- Modules are very actively developed by the community, and the status field in the module documentation indicates the current status.
  - **stableinterface:** the module is stable and safe to use.
  - **preview:** the module is in tech preview and its keywords may change.
  - **deprecated:** the module should not be used anymore, and will be removed in a feature release.
  - **removed:** the module has been removed, and the documentation only still exists to help users migrating to its replacement.

#### Understanding Module Support Status

The supported_by field in ansible-doc indicates who is responsible for supporting a module.
    - **core:** the module is supported by the core Ansible developers.
    - **curated:** the primary support responsibility is with partners and companies in the community. Proposed changes are reviewed by the core developers.
    - **community:** support is completely within the community.

### 3.4 Introducing Essential Ansible Modules

#### Essential Ansible Module

- **ping:** verifies the ability to log in and that Python has been installed.
  - **ansible all -m ping**
- **service:** checks if a service is currently running.
  - **ansible all -m service -a "name=httpd state=started"**
  - **ansible all -m service -a "name=sshd state=started"**
- **command:** runs any command, but not through a shell.
  - **ansible all -m command -a "/sbin/reboot -t now"**
  - **ansible all -m ping**
- **shell:** runs arbitrary commands through a shell.
  - **ansible all -m shell -a set**
- **raw:** runs a command on a remote host without a need for Python.
- **copy:** copies a file to the managed host.
  - **ansible all -m copy -a 'content="hello world" dest=/etc/motd'**
  - **ansible all -m shell -a 'cat /etc/motd'**

### Lesson 3 Lab: Using Ad Hoc Commands

- Use the **ping** module in an ad hoc command to verify that all of your hosts have been set up successfully.
- Use the appropriate ad hoc command to install Python on a host you want to manage.

### Lesson 3 Lab Solution: Using Ad Hoc Commands

```bash
ssh root@snsible3;
exit;

vi inventory;
ansible3.example.com # Add this at the very first line.

save and close the file.
ansible all -m ping;
ansible-doc raw;

ansible -u root -i inventory ansible3.example.com --ask-pass -m raw -a 'yum install -y python3'

ansible all -m ping;
```
