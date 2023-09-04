# Lesson 1 Installing Ansible

- [Lesson 1 Installing Ansible](#lesson-1-installing-ansible)
    - [1.1 Understanding Ansible](#11-understanding-ansible)
    - [1.2 Host Requirements](#12-host-requirements)
    - [1.3 Installing Ansible on the Control Node](#13-installing-ansible-on-the-control-node)
      - [Installing Ansible](#installing-ansible)
      - [Using Python pip to Install Ansible](#using-python-pip-to-install-ansible)
      - [Installing Ansible from the Repositories](#installing-ansible-from-the-repositories)
      - [If using CentOS 8](#if-using-centos-8)
      - [Setting up the Required Environment](#setting-up-the-required-environment)
    - [1.4 Preparing Managed Nodes](#14-preparing-managed-nodes)
      - [Preparing Managed Nodes - 1](#preparing-managed-nodes---1)
      - [Preparing Managed Nodes - 2](#preparing-managed-nodes---2)
    - [1.5 Verifying Ansible Installation](#15-verifying-ansible-installation)
    - [Lesson 1 Lab: Installing Ansible](#lesson-1-lab-installing-ansible)
    - [Lesson 1 Lab Solution: Installing Ansible](#lesson-1-lab-solution-installing-ansible)

### 1.1 Understanding Ansible

- Ansible is a Configuration Management System.
- Ansible will be installed on the Ansible Control Node.
- Connect to the other servers using SSH (Secure-shell).
- Create a playbook on the control node.
- Requirements.
  - Python - Needs to be installed on all nodes, workstation and servers. 
- Playbook generate a python scripts and executed on all nodes.

### 1.2 Host Requirements

- Master node (control.example.com)
  - Install python.
  - Install Ansible.
  - SSH Keygen.
  - A dedicated user account (ex: ansible) 
  - Static IP Address.
- Worker nodes (ansible[num].example.com)
  - Install Python.
  - sudo configuration.
  - Static IP Addresses.
- Configure hosts in /etc/hosts file.
- EPEL - Extra packages for Enterprise Linux.

### 1.3 Installing Ansible on the Control Node

#### Installing Ansible

- Ansible can be installed from the repositories, or using the Python pip installer.
- In this procedure we will discuss Python pip as well as the repositories method.

#### Using Python pip to Install Ansible

```bash
yum install -y python3 # On control.example.com node
yum -y install python3-pip;
alternatives --set python /usr/bin/python3;
su - ansible; pip3 install ansible --user;
ansible --version;
```

#### Installing Ansible from the Repositories

- On the control node: use **subscription-manager repos --list** to verify the name of the latest available repository.
- Use **subscription-manager repos --enable=ansible-2-for-rhel-8-x86_64-rpms** to enable the repository.
- Use **yum -y install ansible** to install the Ansible3 software.
- ansible --version to verify.
- On managed nodes: **yum -y install python3**

#### If using CentOS 8

- On CentOS 8, the Ansible software is expected to be available from the EPEL repository.
- Use **yum install -y epel-release** to make this repository available.
- Use **yum install -y python3 ansible** to install the ansible software.

#### Setting up the Required Environment

After installing Ansible, you will need to set up a dedicated Ansible user account.
  1. On all nodes: **useradd ansible**
  2. **echo password | passwd --studin ansible**;
  3. **echo "ansible ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/ansible**
  4. On control.example.com: su - ansible; ssh-keygen;
  5. **ssh-copy-id ansible1.example.com; ssh-copy-id ansible2.example.com**

### 1.4 Preparing Managed Nodes

#### Preparing Managed Nodes - 1

- Ansible needs a dedicated non-root user account with sudo privileges that can SSH into the managed nodes without entering a password.
- In this course we will use an account with the name ansible to do so.
- to set this account, the following steps have already been completed.
  - **useradd ansible**
  - **echo password | passwd --studin ansible**;
  - **echo "ansible ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/ansible**

#### Preparing Managed Nodes - 2

- On the control host, generate an SSH key and copy over to the managed nodes.
  - ssh-keygen
  - ssh-copy-id ansible1.example.com
- Install Python3
  - yum install -y python3
  - alternatives --set python /usr/bin/Python3

### 1.5 Verifying Ansible Installation

- As user **ansible**, type** ansible --version**;
- Further verification is not possible yet, as additional settings need to be done.

### Lesson 1 Lab: Installing Ansible

- Set up the basic installation of Ansible as described in this lesson.

### Lesson 1 Lab Solution: Installing Ansible

- Follow the above docs to complete the lab.
