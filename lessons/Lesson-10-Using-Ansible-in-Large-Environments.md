# Lesson 10 Using Ansible in Large Environments

- [Lesson 10 Using Ansible in Large Environments](#lesson-10-using-ansible-in-large-environments)
    - [10.1 Managing Inventory](#101-managing-inventory)
      - [Understanding Inventory Options](#understanding-inventory-options)
      - [Managing Dynamic Inventory](#managing-dynamic-inventory)
      - [Writing Dynamic Inventory Scripts](#writing-dynamic-inventory-scripts)
      - [Using Multiple Inventory Files](#using-multiple-inventory-files)
    - [10.2 Addressing Host Patterns](#102-addressing-host-patterns)
      - [Addressing Hosts](#addressing-hosts)
      - [Addressing Hosts Wildcard Examples](#addressing-hosts-wildcard-examples)
    - [10.3 Configuring Parallelism](#103-configuring-parallelism)
      - [Understanding Processing Order](#understanding-processing-order)
      - [Managing Rolling Updates](#managing-rolling-updates)
    - [10.4 Organizing Directory Structure](#104-organizing-directory-structure)
      - [Understanding Inclusion](#understanding-inclusion)
      - [Including Tasks Files](#including-tasks-files)
      - [When to Include Task Files](#when-to-include-task-files)
      - [Using Variables fo fExternal Plays and Tasks](#using-variables-fo-fexternal-plays-and-tasks)
    - [Lesson 10 Lab: Using Ansible in Large Environments](#lesson-10-lab-using-ansible-in-large-environments)
    - [Lesson 10 Lab Solution: Using Ansible in Large Environments](#lesson-10-lab-solution-using-ansible-in-large-environments)

### 10.1 Managing Inventory

#### Understanding Inventory Options

- A static inventory file can be used as a list of managed hosts.
- Dynamic inventory can automatically discover hosts, by talking to an external host management system, such as FreeIPA, Active Directory, Red Hat Satellite and more.
- Also, multiple inventories can be used, for instance by putting multiple inventory files in a directory and use that as the source of inventory.

#### Managing Dynamic Inventory

- Dynamic inventory scripts are available for different environments.
  - Check **https://github.com/ansible/ansible/tree/devel/contrib/inventory**
- The are used like static inventory files, through ansible.cfg or using the -i option to the **ansible[-playbook]** command.
- Instead of using community dynamic inventory scritps,you can also write your own.

#### Writing Dynamic Inventory Scripts

- The only requirement is that the script returns the inventory information in JSON format.
- To see the correct output format, use **ansible-inventory --list** on an inventory.
- Scripts can be written in any language, but Python is common.

```bash
ansible-inventory --list;
cd lesson10
vim pascal.py;
./pascal.py --list;
```

#### Using Multiple Inventory Files

- If the inventory specified is a directory, all inventory files in that directory are consideered.
- This includes static as well as dynamic inventory.
- Inventory files cannot ve created with dependencies to other inventory files.

### 10.2 Addressing Host Patterns

#### Addressing Hosts

- By default, hosts are addressed with their host name as specified in inventory.
- IP addresses can also be used.
- Host groups are common and are defined in inventory.
  - Group **all** is implicit and doen't have to be  defined.
  - Group **ungrouped** is also implicit and addresses all hosts that are not members of a group.
- Wildcards can also be used, **- hosts: '*'** is equivalent to **- hosts: all**
- If special characters are used, always put them between quotes to avoid the special character being interpreted by the shell.

#### Addressing Hosts Wildcard Examples

- **hosts: '*.example.com'**
- **hosts: '192.168.*'**
- **hosts: 'web*'**
- **hosts: web1,db1,192.168.1.10**
- **hosts: web,&eastcoast**
- **hosts: web,!web1**
- **hosts: all,!web**

### 10.3 Configuring Parallelism

#### Understanding Processing Order

- Plays are executed in order on all host referred to, and normally Ansible will start the next task if this task successfully completed on all managed hosts.
- Ansible can run on multiple managed hosts simultaneously, but by default the maximum number of simultaneous hosts is limited to five.
- Set **forks = n** is ansible.cfg to change the maximum number of simultaneous hosts.
- Alternatively, use **-f nn** to specify the max number of forks as argument to the **ansible[-playbook]** command.
- The default of 5 is very limited, so you can set this parameter much higher, in particular if most of the work is done on the managed hosts and not on the control node.

#### Managing Rolling Updates

- The default behavior of running one task on all hosts, and next proceed to the next task means that in cluster environmetns you may have all hosts temporarily deing unavaiable.
- Use the **serial** keyword in the playbook to run hosts through the entire play in batches.

### 10.4 Organizing Directory Structure

#### Understanding Inclusion

- If playbooks grow larger, it is common to use modularity by suing includes and imports.
- Includes and imprts can happen for playbooks as well as tasks.
- An include is a dynamic process; Ansible processes the contents of the included files at the moment that this import is reached.
- An import is a static prcess; Ansible preprocesses the imported file contents before the actual play is started.
  - Playbook imports must be defined at the beginning of the playbook, using **import_playbook**

#### Including Tasks Files

- A task file is  flat list of tasks.
- Use **import_tasks** to statically import as task file in the playbook, it will be included at the location where it is imported.
- Use **include_tasks** to dynamically include a task file.
- Dynamically including tasks means that some features are not available.
  - **ansible-playbook --list-tasks** will not show the tasks.
  - **ansible-playbook --start-at-task** doesn't work.
  - You cannot trigger a handler in an imported task file from the main task file.
- Best practice: store task files in a dedicated directory to make management easier.

#### When to Include Task Files

- When modularity is required, for instance fo differentiate between groups of tasks that need to be executed against specific host types.
- When different groups of IT staff are responsible for different setup tasks.
- If a task needs to be executed only in specific cases.

#### Using Variables fo fExternal Plays and Tasks

- In the design, it is recommended to keep include files as generic as possible.
- Define variables independently from the palybook.
  - In separate include files.
  - Using **group_vars** and **host_vars**
  - Or using local facts
- This allows you to process different values on different groups of hosts, while still using the same playbook.

```bash
vim includes.yml;
vim install-and-setup.yml;
ansible-playbook includes.yml;
```

### Lesson 10 Lab: Using Ansible in Large Environments

- Analyze the project in the lesson10/lab directory. Optimize it for use in a large environment.

### Lesson 10 Lab Solution: Using Ansible in Large Environments

```bash
cd lesson10/lab
ls;
vim lab-playbook.yml;
vim lab-copy-facts.yml;
cd lab-vars/
ls;
vim facts-vars.yml;
# update the below content.
remote_dir: /etc/ansible/facts.d
facts_file: custom.fact

save and close the file.
cd ..
vim lab-playbook.yml;

ansible-playbook lab-copy-facts.yml
ansible -i lab-inventory -C lab-copy-facts.yml;

vim ansible.cfg # change inventory = inventory to inventory = lab-inventory
vim lab-playbook.yml;
ansible-playbook lab-playbook.yml;
```

- Note: Go through the docs again if you run into any error while executing the playbook.
