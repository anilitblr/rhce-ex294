# Lesson 8 Using Ansible Roles

- [Lesson 8 Using Ansible Roles](#lesson-8-using-ansible-roles)
    - [8.1 Understanding Directory Structure Best Practices](#81-understanding-directory-structure-best-practices)
      - [Organizing Ansible Contents](#organizing-ansible-contents)
      - [Directory Layout Best Practices](#directory-layout-best-practices)
    - [8.2 Understanding Ansible Roles](#82-understanding-ansible-roles)
      - [Understaning Roles](#understaning-roles)
      - [Understanding Roles Default Structure](#understanding-roles-default-structure)
      - [Understanding Role Variables](#understanding-role-variables)
      - [Understanding Role Location](#understanding-role-location)
      - [Using Roles in a Playbook](#using-roles-in-a-playbook)
      - [Defining Rle Variables](#defining-rle-variables)
    - [8.3 Using Ansible Galaxy for Standard Roles](#83-using-ansible-galaxy-for-standard-roles)
      - [Using Ansible Galaxy](#using-ansible-galaxy)
    - [8.4 Using the Ansible Galaxy Command Line Tool](#84-using-the-ansible-galaxy-command-line-tool)
      - [Using the Galaxy CLI Utility](#using-the-galaxy-cli-utility)
      - [Managing Roles](#managing-roles)
      - [Using a Requirements File](#using-a-requirements-file)
    - [8.5 Creating Custom Roles](#85-creating-custom-roles)
      - [Creating Roles](#creating-roles)
      - [Creating Roles - Best Practices](#creating-roles---best-practices)
      - [Defining Role Dependencies](#defining-role-dependencies)
      - [Using Conditional Roles](#using-conditional-roles)
    - [8.6 Managing Order of Execution](#86-managing-order-of-execution)
      - [Understanding Role Order of Execution](#understanding-role-order-of-execution)
    - [8.7 Understanding RHEL System Roles](#87-understanding-rhel-system-roles)
      - [Understanding RHEL System Roles](#understanding-rhel-system-roles)
    - [8.8 Using Collections](#88-using-collections)
      - [Understanding Collections](#understanding-collections)
      - [Understanding Collections Names](#understanding-collections-names)
      - [Using Collections in Playbooks](#using-collections-in-playbooks)
      - [Using Collections](#using-collections)
      - [Getting Collection Information](#getting-collection-information)
      - [Listing Collections](#listing-collections)
    - [Lesson 8 Lab: Using Ansible Roles](#lesson-8-lab-using-ansible-roles)
    - [Lesson 8 Lab Solution: Using Ansible Roles](#lesson-8-lab-solution-using-ansible-roles)

### 8.1 Understanding Directory Structure Best Practices

#### Organizing Ansible Contents

- When working with Ansible, it's recommended to use project directories so that contents can be organized in a consistent way.
- Each project directory may have its own ansible.cfg, inventory as well as playbooks.
- If the directory grows bigger, variable files and other include files may be used.
- And finally, roles can be use to standardize and easily re-use specific parts of Ansible.
- For now, consider a role a complete project dedicated to a specific task that is going to be included in the main playbook.

#### Directory Layout Best Practices

- Ansible Documentation describes best practices
  - (https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- Some highlights.
  - On top in the directory, use site.yml as the master playbook.
  - From site.yml, call specific playbooks for specific types of host (webservers.yml, dbservers.yml, etc...)
  - Consider using different inventory files to differentiate between production and staging phases.
  - Use group_vars/ and host_vars/ to set host related variables.
  - Use role to standardize common tasks.

### 8.2 Understanding Ansible Roles

#### Understaning Roles

- Ansible Playbooks can be very similar: code used in one playbook can be useful in other playbooks also.
- To make it easy to re-use code, roles can be used. A role is a collection of tasks, variables, files, templates and other resources in a fixed directory structure that, as such, can easily be included from a playbook.
- Roles should be written in a generic way, such that play specifics can be defined as variables in the play, and overwrite the default variables that should be set in the role.
- Using Roles makes working with large projects more manageable.

```bash
cd playbooks/lesson8/roles/

tree myrole;

myrole/
├── defaults
│   └── main.yml
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml
```

#### Understanding Roles Default Structure

- **defaults** contains default values of role variables. If variables are set at the play level as well, these default values are overwritten.
- **files** may contain static files that are needed from the role tasks
- **handlers** has a main.yml that defines handlers used in the role.
- **meta** has a main.yml that may be used to include role metadata, such as information about author, license, dependencies and more.
- **tasks** contains a main.yml that defines the role task definitaions.
- **templates** is used to store Jinja2 templates.
- **tests** may contain an optional inventory file, as well as a test.yml playbook that can be used to test the role.
- **vars** may contain a main.yml with standard variables for the role (which are not meant to be overwritten by palybook variables)

#### Understanding Role Variables

- Variables can be defined at different levels in a role.
- **vars/main.yml** has the role default variables, which are used in default role functioning. They are not intended to be overwritten.
- **defaults/main.yml** can contain default variables. These have a low precedence, and can be overwritten by variables with the same name that are set in the playbook and which have higher precedence.
- Playbook variables will always overwrite the variables as set in the role. Site-specific variables such as secrets and vault encrypted data should always be managed from the playbook, as role variables are intended to be generic.
- Role variables are defined in the playbook when calling the role and they have the higest precedence and overwrite playbook varialbes as well as inventory variables.

#### Understanding Role Location

- Roles can be obtained in many ways.
  - You can wrirt your own roles.
  - For Red Hat Enterprise Linux, the rhel-system-roles package is available.
  - The community provides roles through the Ansible Galaxy website.
- Role can be stored at a default location, and from there can easily be used from playbooks
  - ./roles has highest precedence
  - ~/.ansible/roles is checked after that
  - /etc/ansible/roles is checked next
  - /usr/share/ansible/roles is checked last

#### Using Roles in a Playbook

- Roles are referred to from playbooks.
- When roles are used, they will run before any task taht is defined in the playbook.

```yaml
---
- name: role demo
  hosts: all
  roles: 
    - role1
    - role2
```

#### Defining Rle Variables

- When calling a role, role variables can be defined.

```yaml
---
- name: role variable demo
  hosts: all
  roles:
    - role: role1
    - role: role2
      var1: cow
      var2: goat
```

### 8.3 Using Ansible Galaxy for Standard Roles

#### Using Ansible Galaxy

- Administrators can define their own roles, or standard roles can be used from Ansible Galaxy.
- Ansible Galaxy is a public website where community provided roles are offered.
- Before writing your own roles, check Galaxy, you may get the roles from there.
- An easy-to-use search interface is available at galaxy.ansible.com

```bash
ansible-galaxy install geerlingguy.nginx;
cd ~/.ansible
ls
cd roles/
tree;
cd geerlingguy.nginx/tasks/
vim main.yml;

cd playbooks/lesson8/
vim nginx-role.yml
ansible-playbook nginx-role.yml;
```

### 8.4 Using the Ansible Galaxy Command Line Tool

#### Using the Galaxy CLI Utility

- **ansible-galaxy search** will search for roles
  - If an argument is provided, **ansible-galaxy** will search for this argument in the role description.
  - Use options **--author**, **--platforms** and **--galaxy-tags** to narrow down the search results.
  - **ansible-galaxy search wordpress --platforms EL**
- **ansible-galaxy info** provides information about roles.
  - **ansible-galaxy info bertvv.wordpress**
- **ansible-galaxy install** downloadds a role and installs it in ~/.ansible/roles
- After download, these roles can be used in playbooks, like any other role.

```bash
ansible-galaxy search 'wordpress' --platforms EL
ansible-galaxy info bertvv.wordpress
ansible-galaxy install bertvv.wordpress

vim wordpress.yml
```

#### Managing Roles

- **ansible-galaxy list** list shows installed roles.
- **ansible-galaxy remove** remove can be used to clean up and remove roles.
- **ansible-galaxy init** creates a directory structure that can be used to start developing your own role.
  - It interacts with the Ansible Galaxy website API.
  - Soecify username and role name as arguments.
  - **ansible-galaxy init user.myrole**

#### Using a Requirements File

- **ansible-galaxy** can be used to install a list of roles based on difinitions in a requirements file.
- A requirements file is a yml file that defines a list of required roles that are specified using the **src** keyword.
- The **src** keyword can contain the name of a role from Ansible Galaxy, or a URL to a custom location pointing to your own roles.
- Create roles/requirements.yml in the projects directory to use it
- Always specify the optional **version** attribute, to avoid getting suprises when a newer version of a role has become available.
```yaml
- src: file:///opt/local/roles/myrole.tar
  name: myrole
  version: 1.0
```
- To install roles using a requirements file, use **ansible-galaxy install -r roles/requirements.yml** 

```bash
vim roles/requirements.yml;
ansible-galaxy install -r roles/requirements.yml -p roles
```

### 8.5 Creating Custom Roles

#### Creating Roles

- To create your own roles, use **ansible-galaxy init myrole** to create the role directory structure.
- Mind the location of the directory structure, you can put it in the local project directory, or in a directory that is accessible for all projects.
- Populate the required role files as discussed before.

#### Creating Roles - Best Practices

- Each role should have its own version control repository.
- Don't put sensitive information in the role, but in the local playbooks or Ansible Vault instead.
- Use **ansible-galaxy init** to create the role structure.
- Don't forget to edit the README.md and the meta/main.yml to contain documentation about your role.
- Roles should be dedicated to one task/function. Use multiple roles to manage multiple tasks/functions
- Have a look at existing (Galaxy) roles before starting to write your own.

#### Defining Role Dependencies

- The meta/main.yml can be used to define role dependencies.
- Dependencies listed here will be installed automatically when this role is used.

#### Using Conditional Roles

- Conditional roles call a role dynamically, using the **include_role** module.
  - This makes it so conditional roles are treated more as tasks.
- Conditional Roles can be combined with conditional statements.
  - This makes it so a role will only run if the conditional statement is true.
  - Use **include_role** in a task statement to do so.

```yaml
---
- hosts: lamp
  tasks:
  - include_role:
      name: lamp
    when: "ansible_facts['os_family'] == 'RedHat'"
```

```bash
cd roles;
tree motd
cd motd;
vim tasks/main.yml;
# Note: explore all *.yml files.
cd ../..
vim motd-role.yml
ansible-playbook motd-role.yml;
ansible ansible2.example.com -m shell -a "cat /etc/motd"
```

### 8.6 Managing Order of Execution

#### Understanding Role Order of Execution

- Role tasks are always executed before playbook tasks.
- Next, playbook tasks are executed.
- And after playbook tasks, handlers are executed.
- Use **pre_tasks** to define playbook tasks that are to be executed before the tasks in a role.
  - If these tasks notify a handler, this handler is executed before as well.
- The **post_tasks** keyword can be used to define playbook tasks that are executed after playbook tasks and roles.
  
```bash
vim pretasks.yml
ansible-playbook pretasks.yml
```

### 8.7 Understanding RHEL System Roles

#### Understanding RHEL System Roles

- The rhel-system-roles package can be installed to provide some standard roles for RHEL 6.10 and later.
- These roles are based on the community linux-system-roles and supported by Red Hat support.
- Use the rhel-system-roles to overcome the difference between configuration choices on RHEL6, 7 and 8
- Lesson 9 is dedicated to working with rhel-system-roles

### 8.8 Using Collections

#### Understanding Collections

- Collections are a new way to bundle Ansible contents to make it more manageable.
- Collections can have different sources, including Ansible community and Red Hat partners.
- Collections may contain the following.
  - **modules** that provide the Ansible core functionality.
  - **roles** that can be included in playbooks for common task execution.
  - **plugins** that extend the Python code on the Ansible control host.

#### Understanding Collections Names

- Collections have a Fully Qualifed Collction Name (FQCN), such as **ansible.netcommon**
- Within this FQCN, plugins, modules etc, are addressed with their own name, such as **ansible.netcommon.cli_command**
- Before collections, you would address just a module name such as **user**, now you address this module as **ansible.builtin.user**
- So using collections involves a lot more typing?
- No! Just start with a **collections** setting in the play header and list all collections you're going to use.

#### Using Collections in Playbooks

- In the play header, the collections keyword can be used.
- It takes a list of collections as its argument.
- After using the **collections** keyword, the collection itself can be addressed the old way: **selinux** instead of **ansible.posix.selinux**
- See **enforce-selinux-simplified.yml** for an example.

```bash
vim enforce-selinux.yml;
vim enforce-selinux-simplified.yml;
ansible-galaxy collection install ansible.posix;
```

#### Using Collections

- In Ansible 2.9, collections are NOT a default port of Ansible.
- To use them, they need to be installed from Ansible Galaxy.
- To install multiple collections, consider using a requirements file.
- **ansible-galaxy collection install -r requirements.yml**
- See collections/requirements.yml for an example.

```bash
vim requirements.yml
```
- In Ansible 2.10 and later, collections are installed by default.

#### Getting Collection Information

- Ansible 2.0 and later contain better collection integration.
  - Collection names can easily be found using **ansible-doc; ansible-doc | grep aws**
- Ansible 2.9 shows only collections that have been installed.
  - Collections are not listed with **ansible-doc -l**
  - Collection documentation is available while addressing collection components directly.
  - Finding collection names only goes through the installed files.

#### Listing Collections

- **ansible --version**
- Use **ansible-galaxy collection install ansible.posix**
- **tree ~/.ansible/collections/ansible_collections/ansible/posix**
- **ansible-doc ansible.posix.selinux; ansible-doc ansible.posix.acl**

### Lesson 8 Lab: Using Ansible Roles

- Create a role that configures an Apache vhost on the managed hosts in the group lamp.
- Have it include a template to create a minimal vhost configuration file in which all references to the hostname are replaced with Ansible facts.
- Use an ad-hoc command to test working of the vhost.

### Lesson 8 Lab Solution: Using Ansible Roles

```bash
vim apache-vhost-role.yml
cd files/html
cat index.html;
cd ../..
ls
cd vhost;
ls
tree;
vim tasks/main.yml;
vim handlers/main.yml;
vim templates/vhost.conf.j2
ansible-playbook apache-vhost-role.yml
ansible lamp -a 'systemctl is-active httpd'
ansible lamp -a 'cat /etc/httpd/conf.d/vhost.conf'
```

**Note:** Lab is not working, fix it.
