# Lesson 4 Getting Started with Playbooks

- [Lesson 4 Getting Started with Playbooks](#lesson-4-getting-started-with-playbooks)
    - [4.1 Using YAML to Write Playbooks](#41-using-yaml-to-write-playbooks)
      - [Why Playbooks](#why-playbooks)
      - [Understanding YAML](#understanding-yaml)
      - [Writing Your First Playbook](#writing-your-first-playbook)
      - [Running Your First Playbook](#running-your-first-playbook)
    - [4.2 Verifying Playbook Syntax](#42-verifying-playbook-syntax)
      - [Optimizing vim for YAML](#optimizing-vim-for-yaml)
      - [Verifying Playbook Syntax](#verifying-playbook-syntax)
    - [4.3 Writing Multiple-Play Playbooks](#43-writing-multiple-play-playbooks)
      - [Understanding Plays](#understanding-plays)
    - [Lesson 4 Lab: Getting Started with Playbooks](#lesson-4-lab-getting-started-with-playbooks)
    - [Lesson 4 Lab Solution: Getting Started with Playbooks](#lesson-4-lab-solution-getting-started-with-playbooks)

### 4.1 Using YAML to Write Playbooks

#### Why Playbooks

- Ad-hoc commands can be used to run one or a few tasks.
- Ad-hoc commands are convenient to test, or when a complete managed infrastructure has not been set up yet.
- Ansible Playbooks are used to run multiple tasks against managed hosts in a scripted way.
- In Playbooks, one or multiple plays are started.
  - Each play runs one or more tasks.
  - In these tasks, different modules are used to perform the actual work.
- Playbooks are writtent in YAML, and have the .yml or .yaml extension.

#### Understanding YAML

- YAML is Yet Another Markup Language according to some.
- According to others it stand for YAML Ain't Markup Language.
- Anyway, it's an easy-to-read format to structure tasks/items that need to be created.
- In YAML files, items are using indentation with white spaces to indicate the structure of data.
- Data elements at the same level should have the same indentation.
- Child items are indented more than the parent items.
- There is no strict requirement about the amount of spaces that should be used, but 2 is common.

#### Writing Your First Playbook

```yaml
---
- name: deploy vsftpd
  hosts: ansible2.example.common
  tasks:
  - name: install vsftpd
    yum: name=vsftpd
  - name: enable vsftpd
    service: name=vsftpd enabled=true
  - name: create readme file
    copy:
      content: "free downloads for everybody"
      dest: /var/ftp/pub/README
      force: no
      mode: 0444
...
```

#### Running Your First Playbook

- Use **ansible-playbook vsftpd.yml** to run the playbook.
- Notice that a successful run requires the inventory and become parameters to be set correctly, and also requires access to an inventory file.
- The output of the **ansible-playbook** command will show what exactly has happened.
Playbooks in general are idempotent, which means that running the same playbook again should lead to the same result.
- Notice there is no easy way to undo changes made by a playbook.

```bash
ansible-playbook vsftpd.yml;
ansible-playbook vsftpd.yml;
```

### 4.2 Verifying Playbook Syntax

#### Optimizing vim for YAML

- In **~/.vimrc**, include the following setting
  - autocmd FileType yaml setlocal ai ts=2 sw=2 et

#### Verifying Playbook Syntax

- **ansible-playbook --syntax-check vsftpd.yml** will perform a syntax check.
- use -v[vvv] to increase output verbosity.
  - **-v** will show task rusults.
  - **-vv** will show task results and task configuration.
  - **-vvv** also shows information about connections to managed hosts.
  - **-vvvv** adds information about plug-ins, users used to run scripts and names of scritps that are executed.
- Use the **-C** option to perform a dry run.
  - **ansible-playbook -C vsftpd.yaml**
  - **ansible-playbook -vvvv vsftpd.yaml**
  - **ansible-playbook -vv vsftpd.yaml**

### 4.3 Writing Multiple-Play Playbooks

#### Understanding Plays

- A play is a series of tasks that are executed against selected hosts from the inventory, using specific credentials.
- Using multiple plays allows running tasks on different hosts, using different credentials from the same playbook.
- Within a play definition, escalation parameters can be defined.
  - **remote_user**: the name of the remote user.
  - **become**: to enable or disable privilege escalation.
  - **become_method**: to allow using an alternative escalation solution.
  - **become_user**: the target user used for provilege escalation.

- vim web-setup-and-test.yml - Update the below content into this file.

```yaml
---
- name: enable webserver
  hosts: ansible1.example.com
  tasks:
  - name: install httpd and firewalld
    yum: 
      name:
        - httpd
        - firewalld
      state: latest
  - name: install welcome page
    copy:
      content: "heelo world"
      dest: /var/www/html/index.html
  - name: start web service
    service:
      - name: httpd
        enabled: true
        state: started
  - name: start firewalld service
    service:
      - name: firewalld
        enabled: true
        state: started
  - name: open firewalld
    firewalld:
      service: httpd
      permanent: true
      state: enabled
      immediate: yes
- name: test webserver access
  hosts: localhost
  become: no
  tasks:
    - name: connect to the web server
      uri:
        url: http://ansible1.example.com
        return_content: yes
        status_code: 200
...
```

```bash
ansible-playbook --syntax-check web-setup-and-test.yml;
ansible-playbook -v web-setup-and-test.yml;
```

### Lesson 4 Lab: Getting Started with Playbooks

- Write a playbook that installs the httpd package on ansible2.example.com
- Ensure that it is started and that the firewall is opened to allow access to it.
- Also create a file /var/www/html/index.html with some welcome text.
- Lastly, write a playbook that will undo all modifications.

### Lesson 4 Lab Solution: Getting Started with Playbooks

- vim install-httpd.yml

```yaml
---
- name: install httpd
  hosts: ansible2.example.com
  tasks:
  - name: install package
    package: 
      name: httpd
      state: present
  - name: create an index.html
    copy:
      content: "Welcoem to this webserver"
      dest: /var/www/html/index.html
  - name: start the service
    service:
      - name: httpd
        enabled: true
        state: started
  - name: open firewall
    firewalld:
      service: http
      permanent: true
      state: enabled
...
```

- Run the playbook.

```bash
ansible-playbook install-httpd.yml;
```

- vi uninstall-httpd.yml

```yaml
---
- name: remove httpd
  hosts: ansible2.example.com
  tasks:
    - name: close firewall
      firewalld:
        service: http
        permanent: yes
        state: disabled
    - name: remove file
      file:
        path: /var/www/html/index.html
        state: absent
    - name: remove package
      package:
        name: httpd
        state: absent
...
```
- Execute ansible playbook to uninstall httpd

```bash
ansible-playbook uninstall-httpd.yml;
ansible ansible2.example.com -a "rpm -qa | grep http"
ansible ansible2.example.com -m shell -a "rpm -qa | grep http"
```
