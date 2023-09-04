# Lesson 12 Managing Software with Ansible
- [Lesson 12 Managing Software with Ansible](#lesson-12-managing-software-with-ansible)
    - [12.1 Understanding Modules Related to Software Management](#121-understanding-modules-related-to-software-management)
      - [Understanding Software Management Tasks](#understanding-software-management-tasks)
      - [Understanding Software Management Modules 1](#understanding-software-management-modules-1)
      - [Understanding Software Management Modules 2](#understanding-software-management-modules-2)
    - [12.2 Implementing a Playbook to Manage Software](#122-implementing-a-playbook-to-manage-software)
    - [Lesson 12 Lab: Managing Software](#lesson-12-lab-managing-software)
    - [Lesson 12 Lab Solution: Managing Software](#lesson-12-lab-solution-managing-software)

### 12.1 Understanding Modules Related to Software Management

#### Understanding Software Management Tasks

- To manage software on RHEL systems, different tasks need to be managed.
- Systems need to be subscribed.
- Repositories and software channels need to be configured.
- Software needs to be installed and removed.

#### Understanding Software Management Modules 1

- **packaage**: Distribution agnostic module to manage packages.
- **win_packages**: Manages packages on Windows.
- **apt**: Manages packages on Ubuntu.
- **yum**: Manages packages on RHEL.
- **yum_repository**: Manages Yum repositories.

#### Understanding Software Management Modules 2

- **package_facts**: Returns information about packages as facts.
- **rpm_key**: Adds or removes GPG keys from an RPM package database.
- **redhat_subscription**: Uses the **subscribtion_manager** command to manage subscriptions.
- **rhn_register**: Managed Red Hat Network registration using **rhnreg_ks**
- **rhn_channel**: Manages RHN Channel subscription.
- Other modules are available, see (https://docs.ansible.com/ansible/latest/modules/list_of_packaging_modules.html) for more information.


### 12.2 Implementing a Playbook to Manage Software

```bash
cd lesson12/lab

vim setup_repo_client.yml
ansible-playbook setup_repo_client.yml;
ansible ansible2.example.com -a 'cat /etc/yum.repos.d/lesson12.repo'
```

### Lesson 12 Lab: Managing Software

- Set up a repository on control.example.com. This repository should offer nultiple files, including the nmap file. Provide a file list using variables.
- Configure ansible1 and ansible2 to use the repository that is provided through the packages repository.
- Install the nmap package from this repository.

### Lesson 12 Lab Solution: Managing Software

```bash
vim setup_repo.yml;
ansible-playbook setup_repo.yml;

vim setup_repo_client.yml;
ansible-playbook setup_repo_client.yml;
```
