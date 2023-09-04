# Lesson 5 Working with Variables and Facts

- [Lesson 5 Working with Variables and Facts](#lesson-5-working-with-variables-and-facts)
    - [5.1 Understanding Variables](#51-understanding-variables)
      - [Understanding Variables](#understanding-variables)
    - [5.2 Using Variables](#52-using-variables)
      - [Defining Variables](#defining-variables)
      - [Using Variables](#using-variables)
    - [5.3 Understanding Variable Precedence](#53-understanding-variable-precedence)
      - [Understanding Variable Scope](#understanding-variable-scope)
      - [Understanding Built-in Variables](#understanding-built-in-variables)
    - [5.4 Managing Host Variables](#54-managing-host-variables)
      - [Using Host Variables](#using-host-variables)
      - [Defining Host Variables Through Inventory](#defining-host-variables-through-inventory)
      - [Using Include Files](#using-include-files)
    - [5.5 Using Multi-valued Variables](#55-using-multi-valued-variables)
      - [Understanding Array and Dictionary](#understanding-array-and-dictionary)
      - [Understanding Dictionary (Hash)](#understanding-dictionary-hash)
      - [Using Dictionary](#using-dictionary)
      - [Using Arrary (list)](#using-arrary-list)
    - [5.6 Using Ansible Vault](#56-using-ansible-vault)
      - [Dealing with Sensitive Data](#dealing-with-sensitive-data)
      - [Creating an Encrypted File](#creating-an-encrypted-file)
      - [Using Playbooks with Vault](#using-playbooks-with-vault)
      - [Managing Vault Files](#managing-vault-files)
    - [5.7 Working with Facts](#57-working-with-facts)
      - [Understanding Ansible Facts](#understanding-ansible-facts)
      - [Managing Fact Gathering](#managing-fact-gathering)
      - [Displaying Fact Names](#displaying-fact-names)
      - [Turning off Fact Gathering](#turning-off-fact-gathering)
    - [5.8 Creating Custom Facts](#58-creating-custom-facts)
      - [Using Custom Facts](#using-custom-facts)
      - [Custom Facts Example File](#custom-facts-example-file)
    - [Lesson 5 Lab: Working with Variables and Facts](#lesson-5-lab-working-with-variables-and-facts)
    - [Lesson 5 Lab Solution: Working with Variables and Facts](#lesson-5-lab-solution-working-with-variables-and-facts)

### 5.1 Understanding Variables

#### Understanding Variables

- A variable is a label that is assigned to a specific value to make it easy to refer to that value throught the playbook.
- Variables can be defined by administrators at different levels.
- A fact is a special type of variable, that referes to a current stte of an Ansible managed system.
- Variables are particularly useful when deailing with managed hosts where specifics are different.
  - Set a variable web_service on Ubuntu and Red Hat.
  - Refer to the variable web_service instead of the specific service name.

### 5.2 Using Variables

- Variables can be set at different levels.
  - In a playbook.
  - In inventory (deprecated)
  - In inclusion files
- Variable names have some requirements.
  - The name must start with a letter.
  - Variable names can only contain letters, numbers and underscores.

#### Defining Variables

- Variables can be defined in a vars block in the beginning of a playbook.

    ```yaml
    ---
    - hosts: all
    vars:
        web_package: httpd
    ```

- Alternatively, variables can be defined in a variable file, which will be included from the playbook.

    ```yaml
    ---
    - hosts: all
    vars_files:
        - vars/users.yml
    ```

#### Using Variables

- After defining the variables, it can be used later in the playbook.
- Notice that order does matter.
- Refer to a variable as {{ web_package }}
- In conditional statements (discussed later), no curly braces are needed to refer to variable values.
  - **when: "'not found' in command_result.err"**
  - {% if ansible_facts['devices']['sdb'] is defined %} 
        Secondary disk size: {{ ansible_facts['devices]['sdb]['size']}}
- If the variable is the first element, using quotes is mandatroy.
- vi user.yml

```yaml
---
- name: create a user using a variable
  hosts: all
  vars:
    user: anil
  tasks:
    - name: create a user {{ user }}
      user:
        name: "{{ user }}"
...
```

- Execute a palybook.

    ```bash
    ansible-playbook user.yml
    ansible all -m shell -a "grep anil /etc/passwd"
    ```
### 5.3 Understanding Variable Precedence

#### Understanding Variable Scope

- Variables can be set with different types of scope.
  - Global scope: this is when a variable is set from inventory or the command line.
  - Play scope: this is applied when it is set from a play.
  - Host scope: this is applied when set in inventory or using a host variable inclusion file.
- When the same variable is set at different levels, the most specific level gets precedence.
- When a variable is set from the command line, it will overwrite anything else.
  - **ansible-playbook site.yml -e "web_package=apache"**

#### Understanding Built-in Variables

- Some variables are built in and cannot be used for anything else.
  - **hostvars**
  - **inventory_hostname**
  - **inventory_hostname_short**
  - **groups**
  - **ansible_check_mode**
  - **ansible_play_batch**
  - **ansible_play_hosts**
  - **ansible_version**

### 5.4 Managing Host Variables

#### Using Host Variables

- Variables can be assigned to hosts and to groups of hosts.
- The old way of doing so is through inventory, but this is now deprecated because if mixed two types of information in one file.
- Instead, you should use directories to populate host and group variables.

#### Defining Host Variables Through Inventory

- A variable can be assigned directly to a host.
    ```inventory_file
    [servers]
    web1.example.com web_package=httpd
    ```
- Alternatively, variables can be set for host groups.
    ```inventory_file
    [servers:vars]
    web_package=httpd
    ```

#### Using Include Files

- To define host and host group variable, directories should be created in the current project directory.
- Use **~/myproject/host_vars/web1.example.com** to include host specific variables.
- Use **~/myproject/group_vars/webservers** to include host group specific variables.
- Notice that you do not have to define variable in these directories in the playbook, they are picket up automatically.

  ```bash
  cd playbooks/lesson5/webservers/
  sudo yum install -y tree;
  tree;
  ansible-playbook site.yml;
  ```

### 5.5 Using Multi-valued Variables

#### Understanding Array and Dictionary

- Multi-valued variables can be used in playbooks.
- When using a multi-valued variable, it can be written as an array (list), or as a directory (hash).
- Each of these has their own specific use cases.

#### Understanding Dictionary (Hash)

- Dictionaries can be written in two ways.

  ```yaml
  ---
  users:
    anil:
      username: anil
      shell: /bin/bash
    kumar:
      username: kumar
      shell: /bin/bash
  ```

- Or as

  ```yaml
  ---
  users:
    anil:{username:'anil', shell:'/bin/bash'}
    kumar:{username:'kumar', shell:'/bin/bash'}
  ```

#### Using Dictionary

- To address items in a dictionary, you can use two notations.
  - variable_name['key'], as in users['anil']['shell]
  - variable_name.key, as in users.anil.shell
- You cannot use **loop** or **with_items** on dictionaries.

#### Using Arrary (list)

- Array provide a list of items, where each item can be addressed separately.

  ```yaml
  ---
  users:
  - username: anil
    shell: /bin/bash
  -username: kumar
    shell: /bin/sh
  ```
- Individual items in the arrary can be addressed, using the {{ var_name[0] }} notation.
- To access all variables, you can use **with_items** or **loop**

  ```bash
  ansible-playbook multi-list.yml
  ansible-playbook multi-dictionary.yml
  ```

### 5.6 Using Ansible Vault

#### Dealing with Sensitive Data

- Some modules require sensitive data to be processed.
- This may include webkeys, passwords, and more.
- To process sensitive data in a swcure way, Ansible Vault can be used.
- Ansible Vault is used to encrypt and decrypt files.
- To manage this process, the **ansible-vault** command is used.

#### Creating an Encrypted File

- To create an encrypted file, use **ansible-vault create playbook.yml**
- This command will prompt for a new vault password, and opens thte file in **vi** for further editing.
- As an alternative for entering passwords on the prompt, a vault password file may be used, but you will have to make sure this file is protected in another way: **ansible-vault create --vault-password-file=vault-pass playbook.yml**
- To view a vault encrypted file, use **ansible-vault view playbook.yml**
- To edit, use **ansible-vault edit playbook.ymml**
- Use **ansible-vault encrypt playbook.yml** to encrypt an existing file, and use **ansible-vault decrypt playbook.yml** to decrypt it.
- To change a password on an existing file, use **ansible-vault rekey**

#### Using Playbooks with Vault

- To run a playbook that access Vault encrypted files, you need to use the **--vault-id @prompt** option to be prompted for a password.
- Alternatively, you can store the password as a single-line string in a password file, and access that using the **--vault-password-file=vault-file** option

#### Managing Vault Files

- When setting up projects with Vault encrypted files, it makes sense to use separate files to store encrypted and non-encrypted variables.
- To store host or host-group related variables files, you can use the following structure.
  ```yaml
  -group_vars
    --dbservers
      - vars
      - vault
  ```
  - This replaces the solution that was discussed earlier, where all variables are stored in a file with the name of the host or host group.

  ```bash
  ansible-vault create secret.yml
  New Vault password: xxxxxxxx
  Confirm New Vault password: xxxxxxxx

  - Update the file.
  username: anil
  pwhash: password

  save and close the file.

  cat secret.yml

  ansible-playbook --ask-vault-pass create-user.yml
  Vault password: xxxxxxxxxx

  ehco password > vault-pass
  chmod 600 vault-pass;
  ansible-playbook --vault-password-file=vault-pass create-user.yml;
  ```

### 5.7 Working with Facts

#### Understanding Ansible Facts

- Ansible facts are variable that are automatically set and discovered by Ansible on managed hosts.
- Facts contain information about hosts that can be used in conditionals.
- For instance, before installing specific software you can check that a managed host runs a specific kernel version.

#### Managing Fact Gathering

- By default, all playbooks perform fact gathering before running the actual plays.
- You can run fact gathering in an ad hoc command using the **setup** module.
- To show facts, use the debug module to print the value of the ansible_facts varibale.
- Notice that in facts, a hierarchical relationship is shown where you can use the dotted format to refer to a specific fact.

  ```bash
  ansible -m setup all;
  cat facts/facts.yml;
  ansible-playbook facts/facts.yml

  cat facts/ipfact.yml;
  ansible-playbook facts/ipfact.yml
  ```

#### Displaying Fact Names

- In Ansible 2.4 and before, Ansible facts were stored as individual variables, such as **ansible_hostname** and **ansible_interface**
- In Ansible 2.5 and later, all facts are stored in one variable with the name **ansible_facts**, and referring to specific facts happens in a different way: **ansible_facts['hostname']** and **ansible_facts['interface']**
- The old approach is referred to as "injecting facts as variables", and this behavior can be managed through the **inject_facts_as_vars** parameter.

#### Turning off Fact Gathering

- Disabling fact gathering may seriously speed up playbooks.
- Use **gather_facts: no** in the play header to disable.
- Even if fact gathering is disabled, it can be enabled again by running the **setup** module in a task.

### 5.8 Creating Custom Facts

#### Using Custom Facts

- Using facts allow administrators to dynamically generate variables which are stored as facts.
- Custom facts are stored in an ini or json file in the /etc/ansible/facts.d directory on the managed host.
  - The name of these files must end in .fact
- Custom facts are stored in the ansible_facts.ansible_local variable.
- Use **ansible hostname -m setup -a "filter=ansible_local"** to display local facts.
- Notice how fact filename and label are used in the fact.

#### Custom Facts Example File

```ini
[localfacts]
package = vsftpd
service = vsftpd
state = started
```

```bash
cat newlocalfacts.yml;
cat localfacts.fact
ansible-playbook newlocalfacts.yml
ansible all -m setup -a "filter=ansible_local"
```

### Lesson 5 Lab: Working with Variables and Facts

- Create a playbook that will first define local facts on the file servers. These facts should define the name of the smbd service, and the samba package.
- Ensure the first play copies these facts to the file servers.
- Next, in the same playbook, use a second play that will install and enable the smbd service.

### Lesson 5 Lab Solution: Working with Variables and Facts

```bash
vim localfacts.yml;
vim localfacts.fact;
ansible-playbook localfacts.yml;
ansible ansible1.example.com -m shell -a "rpm -ql samba | grep smb"
```
