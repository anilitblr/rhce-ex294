# Lesson 2 Setting up a Managed Environment

- [Lesson 2 Setting up a Managed Environment](#lesson-2-setting-up-a-managed-environment)
    - [2.1 Setting up Static Inventory](#21-setting-up-static-inventory)
      - [Managing Static Inventory](#managing-static-inventory)
      - [Inventory File Locations](#inventory-file-locations)
      - [Static Inventory Example](#static-inventory-example)
      - [Host Groups Usage](#host-groups-usage)
    - [2.2 Understanding Dynamic Inventory](#22-understanding-dynamic-inventory)
      - [Understaning Dynamic Inventory](#understaning-dynamic-inventory)
    - [2.3 Understanding Ansible Configuration Files](#23-understanding-ansible-configuration-files)
      - [Understanding ansible.cfg](#understanding-ansiblecfg)
      - [Connecting to the Remote hosts](#connecting-to-the-remote-hosts)
      - [Escalating Privileges](#escalating-privileges)
      - [Understanding Localhost Connections](#understanding-localhost-connections)
    - [2.4 Managing Ansible Configuration Files](#24-managing-ansible-configuration-files)
    - [Lesson 2 Lab: Setting up an Ansible Managed Environment](#lesson-2-lab-setting-up-an-ansible-managed-environment)
    - [Lesson 2 Lab Solution: Setting up an Ansible Managed Environment](#lesson-2-lab-solution-setting-up-an-ansible-managed-environment)

### 2.1 Setting up Static Inventory

- DNS.
- /etc/hosts
- Inventory
- Inventory can be used to point to specific server.
- Inventory can also be used to point to group of specific servers.
- Static inventory.
- Dynamic inventory for the cloud environment managed nodes.

#### Managing Static Inventory

- In a minimal form, a static inventory is a list of host names and Ip addresses that can be managed by Ansible.
- Hosts can be grouped in inventory to make it easy to address multiple hosts at once.
- A host can be a member of multiple groups.
- Nested groups are also available.
- It is common to work with project-based inventory files.
- Variables can be set from the inventory file - but this is deprecated practice.
- Ranges can be used.
  - server[1:20] matches server1 up to server20
  - 192.168.[4:5].[0.255] matches two full class C subnets.

#### Inventory File Locations

- /etc/ansible/hosts is the default inventory
- Alternative inventory location can be specified through the ansible.cfg configuration file.
- Or use the **-i inventory** option to specify the location of the inventory file to use.
- It is common practice to put the inventory file in the current projecct directory.

#### Static Inventory Example

```bash
[webservers]
web1.example.com
web2.example.com

[fileservers]
file1.example.com
file2.example.com

[servers:children] # Nested group.
webservers
fileservers
```

#### Host Groups Usage

- Functional host groups.
  - web
  - lamp
- Regional host groups
  - europe
  - africa
- Staging host groups
  - test
  - development
  - production

```bash
mkdir install;
cd install;

vim inventory;

[web]
ansible1.example.com

[db]
ansible2.example.com

save and close the file.

ansible -i inventory all --list-hosts;
ansible -i inventory ungrouped --list-hosts;
```

### 2.2 Understanding Dynamic Inventory

#### Understaning Dynamic Inventory

- Dynamic inventory can be used to discover inventory in dynamic environment such as cloud.
- Many community-provided dynamic inventory scripts are available.
- These scripts are referred to in the same way as static inventory, but must have the execute permission set.
- Alternatively, it's relatively easy to write your own dynamic inventory.
- https://docs.ansible.com/ansible/latest/inventory_guide/intro_dynamic_inventory.html
- git clone https://github.com/sandervanvugt/rhce8

### 2.3 Understanding Ansible Configuration Files

```bash
vi ansible.cfg

[defaults]
inventory = inventory
remote_user = ansible
host_key_checking = false

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```

#### Understanding ansible.cfg

- Settings in ansible.cfg are organized in two sections.
  - [defaults] sets default settings
  - [privilege_escalation] specifies how Ansible runs commands on manged hosts.
- The following settings are used.
  - **inventory** specifies the path to the inventory file.
  - **remote_user** is the name of the user that logs on the remote hosts.
  - **ask_pass** specifies whether or not to prompt for a password.
  - **become** indicates if you want to automatically switch to the become_user.
  - **become_method** sets how to become the other user.
  - **become_user** specifies the target remote user.
  - **become_ask_pass** sets if a password should be asked for when escalating.

#### Connecting to the Remote hosts

- The default protocol to connect to the remote host is SSH.
  - Key based authentication is the common approach, but password based authentication is possible as well.
  - Use **ssh-keygetn** to generate a public/private SSH key pair, and next use **ssh-copy-id** to copy the public key over to the managed hosts.
  - Other connection methods are available, to manage Windows for instance, use **ansible_connection: winrm** and set **ansible_port:5986**

#### Escalating Privileges

- **sudo** is the default machanism, **su** can also be used but is uncommon.
- A password can be asked for, but it's common to do password-less escalation.
- To set up password-less sudo, create a snapin file /etc/sudoers.d/ansible with the following contents.
  - ansible ALL=(ALL) NOPASSWD: ALL

#### Understanding Localhost Connections

- Ansible has an implicit localhost entry to run Ansible commands on the localhost.
- When connecting to localhost, the default **become** settings are not used, but the account that ran the Ansible command is used.
- Ensure this account has been configured with the appropriate **sudo** privileges.

### 2.4 Managing Ansible Configuration Files

- The ansible.cfg file is considered in order of precedence.
  - **/etc/ansible/ansible.cfg** is the default.
  - **~/.ansible.cfg** if exists will overwrite the default.
  - **./ansible.cfg** is the configuration file in the current Ansible project directory and it will always have precedence if it exists.
- Alternatively, if a variable ANSIBLE_CONFIG exists to refer to a specific config file, this will always have precedence.
- Use **ansible --version** to find which configuration file currently is used.

### Lesson 2 Lab: Setting up an Ansible Managed Environment

- Finalize setting up the Ansible managed environment and ensure your setup meets the following requirements.
  - User Ansible is used for all management tasks on managed hosts.
  - Inventory is set up.
  - Ansible.cfg is configured for sudo provilege escalation.
  - SSH key-based login has been configured.

### Lesson 2 Lab Solution: Setting up an Ansible Managed Environment

- Follow the above docs to complete the lab.
