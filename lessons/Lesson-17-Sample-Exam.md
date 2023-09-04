# Lesson 17 Sample Exam
- [Lesson 17 Sample Exam](#lesson-17-sample-exam)
    - [17.1 Exam Information](#171-exam-information)
      - [Exam Information and Tips](#exam-information-and-tips)
    - [17.2 Configuring the Basic Setup](#172-configuring-the-basic-setup)
      - [Basic Setup](#basic-setup)
    - [17.3 RHCE Sample Exam Assignments](#173-rhce-sample-exam-assignments)
      - [Task 1: Setting up Inventory](#task-1-setting-up-inventory)
      - [Task 2: Setting up a Repository Server](#task-2-setting-up-a-repository-server)
      - [Task 3: Setting up Repositories](#task-3-setting-up-repositories)
      - [Task 4: Configuring Managed Hosts](#task-4-configuring-managed-hosts)
      - [Task 5: Managing File Content](#task-5-managing-file-content)
      - [Task 6: Working with Roles](#task-6-working-with-roles)
      - [Task 7: Creating Playbook - 1](#task-7-creating-playbook---1)
      - [Task 7: Creating Playbook - 2](#task-7-creating-playbook---2)
      - [Task 7: Creating Playbook - 3](#task-7-creating-playbook---3)
      - [Task 8: Installing Roles from Galaxy](#task-8-installing-roles-from-galaxy)
      - [Generating a File](#generating-a-file)
      - [Task 10: Using Encrpted Passwords](#task-10-using-encrpted-passwords)
      - [Task 11: Managing Storage](#task-11-managing-storage)
    - [17.4 Setting up Ansible Solution](#174-setting-up-ansible-solution)
      - [Task 1: Setting up Inventory](#task-1-setting-up-inventory-1)
      - [Task 2: Setting up a Repository Server](#task-2-setting-up-a-repository-server-1)
    - [17.5 Managing File Content Solution](#175-managing-file-content-solution)
      - [Task 5: Managing File Content](#task-5-managing-file-content-1)
    - [17.6 Working with Roles Solution](#176-working-with-roles-solution)
    - [17.7 Creating Playbooks Solution](#177-creating-playbooks-solution)
    - [17.8 Installing Ansible Galaxy Roles Solution](#178-installing-ansible-galaxy-roles-solution)
    - [17.9 Generating a File Solution](#179-generating-a-file-solution)
    - [17.10 Creating a User with an Encrypted Password Solution](#1710-creating-a-user-with-an-encrypted-password-solution)
    - [17.11 Managing Storage Solution](#1711-managing-storage-solution)

### 17.1 Exam Information

#### Exam Information and Tips

- The RHCE EX294 exam is 4 hours.
- Timing is an issue, there's a lot of work to do.
- Some questions are simple, some require a complex answer. Make sure to pick the low-hanging fruit first.
- The exam environment is NOT commnected to the Internet.
- The Ansible documentation website is available on the exam.
- Use ansible-doc to find examples of YAML code.
- Consider creating a base YAML file that can be used while working through the assignments.
- Don't forget about Ansible Galaxy and Roles.

### 17.2 Configuring the Basic Setup

#### Basic Setup

- To work through this exam, you need a total of 5 servers, running RHEL 8 or CentOS 8.
  - One server needs to be configured as control host.
  - The other 4 servers are configured as managed servers, using the names ansible1.example.com thru ansible4.example.com. The IP addresses used on the managed servers are not important, you can pick anything that matches your configuration. Make sure the servers meet the following requirements.
    - 1 GB of RAM.
    - 10 GiB disk space on the primary disk /dev/sda
    - 5 GB disk space on the secondary disk /dev/sdb which only exists on ansible1 and ansible3.
    - The root user account has been configured with the password "password" on each of the servers.
    - The control server has a user account "ansible". SSH public and private keys have been generated for this user on control.example.com. No further configuration has been done yet.
    - In the assignments in this exam, you will need to create scripts and yaml files. Make sure that all of these scripts are stored in the directory /home/ansible

### 17.3 RHCE Sample Exam Assignments

#### Task 1: Setting up Inventory

- Configure the control host with a static inventory, as well as the ansible.cfg configurtion file. In the static inventory, configure the following host groups.
  - Group test with ansible1.example.com as a member.
  - Group dev with ansible2.example.com as member.
  - Group prod with ansible3.example.com and ansible4.example.com as a member.
  - A group servers, with groups dev and prod as a member.
  - Ensure tht hosts can be reached through their FQDN, but also by using the short name (so: ansible1.example.com as well as ansible1)

#### Task 2: Setting up a Repository Server

- Create a playbook with the name setupreposerver.yml to set up the control host as a repository host. Make sure this host meets the following requirements, which all must be done by the playbook.
  - The RHEL 8 installation ISO must be loop mounted on the directory /var/ftp/repo
  - The firewalld service is disabled
  - The vsftpd service is started as well as enabled, and allows anonymouse user access to the /var/ftp/repo directory.

#### Task 3: Setting up Repositories

- Create a script that configures the managed servers as repository clients to the repository server that you have set up in the previous task. This script must used ad-hoc commands, and perform the following tasks.
  - Disable any currently existing repository.
  - Enable access to the BaseOS repository on control.example.com.
  - Enable access to the AppStream repository on control.example.com.

#### Task 4: Configuring Managed Hosts

- Create a script with the name setuphosts.sh that uses ad hoc commands to complete configuration on the managed servers. This includes.
  - Installing Python.
  - Creating a user with the name ansible.
  - Creating a sudo configuration that allows user ansible to run tasks with root privileges.
  - As the last command in this script, an ad-hoc command should be used to call the appropriate module to test connectivity to the remote hosts.

#### Task 5: Managing File Content

- Use an automated solution to create the contents of the /etc/hosts file on all managed hosts based on information that was found from the inventory.
  - Tip! Use Templates and Ansible Facts.

#### Task 6: Working with Roles

- Create a custom role to configure SSH. Make sure that it contains the following elements.
  - A template file for the sshd_config, where the SSH port, AllowedUsers, and PermitRoot parameters are managed through variable (which must be set when calling the role from the playbook)
- Create a playbook that calls thr role. Before calling the role, the playbook should ensure that SELinux is in enforcing state. Further, the playbook must defind variables to set the SSH port to 2022, and AllowedUser to the user Ansible.
- Ensure the role is only called with the OS family is set to RedHat.
- Also ensure that the filewall is opened, and the appropriate SELinux context is set on the port.

#### Task 7: Creating Playbook - 1

- Create a playbook to set up ansible1.example.com as a web server. Use another playbook to install wget and elinks on ansible2 and ansible3. for the playbooks, use the following instructions.
  - Create a host group webservers and a host group webclients to run the playbooks on the appropriate hosts.
  - Provide a sample vhost.conf file as Jinja2 template. Copy it to the web/templates directory and ensure it is copied to all web servers. Use variable and/or facts to ensure the local hostname is used in the name of the vhost file, as well as all references to the local host.
  - Defind the following variables and use them while setting up the web servers.
    - web_package httpd
    - web_service httpd
    - web_config_file /etc/httpd/conf/httpd.conf
    - firewall_service http

#### Task 7: Creating Playbook - 2

- Ensure that the web server package is installed.
- Ensure that the service is enabled and started.
- Open a port in the firewalld.
- Copy the httpd.conf template file, using the following properties.
  - owner: root
  - group: root
  - permission mode: 0644
  - SELinux context: httpd_config_t
- Use a handler to restart the web server after copying the configuration file.

#### Task 7: Creating Playbook - 3

- Create a playbook that sets up the web-client machines and installs curl and wget on them.
- Create a generic playbook with the name "web" which you can use to run both playbooks by using includes or imports.

#### Task 8: Installing Roles from Galaxy

- Write a playbook with the name galaxyrole.yml. This playbook should meet the following requirements.
  - Use a requirements file to install the **geerlingguy.redis** and **geerlingguy.nginx** roles.
  - Make sure the current version of the role is installed, but if a newer version becomes available, this will NOT automatically be installed.
- Configure the playbook such that after running it, redis as well as nginx are available on ansible4.example.com

#### Generating a File

- Write a playbook that generate the motd file using a template. The motd file should write a welcome message, and a small report that lists the following itesm.
  - System memory.
  - Processor count.
  - Primary disk size.
  - Secondary disk size.
- The playbook should use facts to fill in the appropriate values. If the secondary disk is not available, the secondary disk size should be set to NOT AVAILABLE.

#### Task 10: Using Encrpted Passwords

- Write a playbook with the name userpw.yml that creates a user list with an AES256 encoded password. Make sure this password is included from an Ansible Vault encrypted file.

#### Task 11: Managing Storage

- Write a playbook with the name createvg.yml. It should create a 1GiB partition /dev/sdb1 on ansible1 and a 3GiB partition /dev/sdb1 on ansible3. Next, it should setup a volume group on all hosts on which the disk /dev/sdb is defined.
- Write a playbook setuplv.yml that configures an LVM volume with the name lvdata in the vgdata volume group. The size of the volume must be 2000 MiB and the playbook should run on all servers. If the volume group vgdata does not exist, the playbook must show the message "vgdata" does not exist. If the requested size is not available, the playbook must return the message "insufficient size".

### 17.4 Setting up Ansible Solution

#### Task 1: Setting up Inventory
  - Configure the control host with a static inventory, as well as the ansible.cfg configurtion file. In the static inventory, configure the following host groups.
    - Group test with ansible1.example.com as a member.
    - Group dev with ansible2.example.com as member.
    - Group prod with ansible3.example.com and ansible4.example.com as a member.
    - A group servers, with groups dev and prod as a member.
    - Ensure tht hosts can be reached through their FQDN, but also by using the short name (so: ansible1.example.com as well as ansible1)

```inventory
vim inventory
# Add the below content.
control.example.com
ansible1.example.com
ansible2.example.com
ansible3.example.com
ansible4.example.com

[test]
ansible1.example.com

[dev]
ansible2.example.com

[prod]
ansible3.example.com
ansible4.example.com

[servers:children]
dev
prod
```

#### Task 2: Setting up a Repository Server

- Create a playbook with the name setupreposerver.yml to set up the control host as a repository host. Make sure this host meets the following requirements, which all must be done by the playbook.
  - The RHEL 8 installation ISO must be loop mounted on the directory /var/ftp/repo
  - The firewalld service is disabled
  - The vsftpd service is started as well as enabled, and allows anonymouse user access to the /var/ftp/repo directory.

```bash
vim lesson12/setup_repo.yml

ansible ansible3.example.com -u root --ask-pass -m user -a name=ansible

ansible ansible3.example.com -u root --ask-pass -m authorized_key -a "user=ansible state=present key=\"{{ lookup(\'file\',\'/home/ansible.ssh/id_rsa.pub\') }}\""

ansible ansible3.example.com -u root --ask-pass -m copy -a "content='ansible ALL=(ALL) NOPASSWD: ALL' dest=/etc/sudoers.d/ansible mode=0440" 
```

### 17.5 Managing File Content Solution

#### Task 5: Managing File Content

- Use an automated solution to create the contents of the /etc/hosts file on all managed hosts based on information that was found from the inventory.
  - Tip! Use Templates and Ansible Facts.

```bash
cd lesson7/
vim hostsfile.yml
vim templates/hosts.j2 
```

### 17.6 Working with Roles Solution

```bash
cd lesson17/task4/roles/ssh-setup
tree;
```

### 17.7 Creating Playbooks Solution

```bash
```

### 17.8 Installing Ansible Galaxy Roles Solution

```bash
cd lesson8/
vim roles/requirements.yml
ansible-galaxy install -r roles/requirements.yml
ansible-galaxy install -r roles/requirements.yml -p somewhere # somewhere means directory path to install roles.
```

### 17.9 Generating a File Solution

```bash
cd lesson17/
vim motd.j2
vim task9.yml
ssh ansible2; # The motd file content should show as login message.
```

### 17.10 Creating a User with an Encrypted Password Solution

```bash
cd lesson13/
vim userpw.yml
```

### 17.11 Managing Storage Solution

```bash
vim createvg.yml
vim setuplv.yml
```
