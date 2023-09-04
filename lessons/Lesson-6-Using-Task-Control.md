# Lesson 6 Using Task Control

- [Lesson 6 Using Task Control](#lesson-6-using-task-control)
    - [6.1 Using Loops and Items](#61-using-loops-and-items)
      - [Understanding Loops](#understanding-loops)
      - [Using Variables to Define a Loop](#using-variables-to-define-a-loop)
      - [Using Hashes/Dictionaries in Loops](#using-hashesdictionaries-in-loops)
      - [Understand loops vs. items](#understand-loops-vs-items)
    - [6.2 Using Register Variables with Loops](#62-using-register-variables-with-loops)
      - [Understanding register](#understanding-register)
    - [6.3 Using when to Run Tasks Conditionally](#63-using-when-to-run-tasks-conditionally)
      - [Using Conditions](#using-conditions)
      - [Defining Simple Conditions](#defining-simple-conditions)
      - [Examples](#examples)
    - [6.4 Testing Multiple Conditions](#64-testing-multiple-conditions)
      - [Combining Loops and Conditions](#combining-loops-and-conditions)
    - [6.5 Using Handlers](#65-using-handlers)
      - [Understanding Handlers](#understanding-handlers)
    - [6.6 Using Blocks](#66-using-blocks)
      - [Understaing Ansible Blocks](#understaing-ansible-blocks)
    - [6.7 Dealing with Failures](#67-dealing-with-failures)
      - [Understaing Failure Handling](#understaing-failure-handling)
      - [Defining Failure States](#defining-failure-states)
    - [6.8 Managing Changed Status](#68-managing-changed-status)
      - [Handling Changed Status](#handling-changed-status)
    - [Lesson 6 Lab: Using Task Control](#lesson-6-lab-using-task-control)
    - [Lesson 6 Lab Solution: Using Task Control](#lesson-6-lab-solution-using-task-control)

### 6.1 Using Loops and Items

#### Understanding Loops

- The **loop** keyword allows you to iterate through a simple list of items.
- Before Ansible 2.5, the **items** keyword was used instead.

```yaml
---
- name: start some services
service:
    name: "{{ item }}"
    state: started
loop:
    - vsftpd
    - httpd
...
```

#### Using Variables to Define a Loop

- The list that loop is using can be defined by a variable.

```yaml
---
vars:
my_services:
    - httpd
    - vsftpd
tasks:
- name: start some services
    service:
    name: "{{ item }}"
    state: started
    loop: "{{ my_services }}"
...
```
#### Using Hashes/Dictionaries in Loops

- Each item in a loop can be a hash/dictionary with multiple keys in each hash/dictionary.

```yaml
---
- name: create users using a loop
hosts: all
tasks:
- name: create users
    user:
    name: "{{ item.name }}"
    state: present
    groups: "{{ item.groups }}"
    loop:
    - name: anil
        groups: wheel
    - name: kumar
        groups: users
...
```
#### Understand loops vs. items

- The **loop** keywork is the current keywork.
- In previous versions of Ansible, the **with_\*** keyword was used for the same purpose.
- This syntax will probably be deprecated in future versions of Ansible.
  - **with_items**: equivalent to the **loop** keywork.
  - **with_file**: the **item** contains a file, which contents is used to loop through.
  - **with_sequence**: generates a list of values based ona numeric sequence.
  
```bash
cd playbooks/lesson6/
vim loopservices.yml;
ansible-playbook loopservices.yml
vim loopusers.yml
ansible-playbook loopusers.yml
```

### 6.2 Using Register Variables with Loops

#### Understanding register

- A **register** is used to store the output of a command and address it as a variable.
- You can next use the result of the command in a conditional or in a loop.

```bash
vim register_loop.yml
ansible-playbook register_loop.yml

vim register_command.yml
ansible-playbook register_command.yml;
```

### 6.3 Using when to Run Tasks Conditionally

#### Using Conditions

- **when** statements are used to run a task conditionally.
- A condition can be used to run a task only if sepcific conditions are true.
- Playbook variables, registered variables, and facts can be used in conditions and make sure that tasks only run if specific conditions are true.
- For instance, check if a task has run successfully, a certain amount of memory is available, a file exist, etc.

#### Defining Simple Conditions

- The simplest example of a condition, is to check whether a Boolean variable is true or false.
- You can also check and see if a non-Boolean variable has a value and use that value in the conditioinal.
- Or use a conditional in which you compare the value of a fact to the specific value of a variable.

#### Examples

- ansible_machine == "x86_64"
- ansible_distribution_major_version == "8"
- ansible_memfree_mb == 1024
- ansible_memfree_mb < 256
- ansible_memfree_mb > 256
- ansible_memfree_mb <= 256
- ansible_memfree_mb != 512
- my_variable is defined
- my_variable is not defined
- my_variable
- ansible_distribution in supported_distros

```bash
vim distro.yml;
ansible-Playbook distro.yml;
```

### 6.4 Testing Multiple Conditions

- **when** can be used to test multiple conditions as well.
- Use **and** or **or** and group the conditions with parentheses.
  - **when: ansible_distribution == "CentOS" or ansible_distribution == "RedHat"**
  - **when: ansible_machine == "x86_64" and ansible_distribution == "CentOS"**
- The **when** keywork also supporta a list, and when using a list, all of the conditions must be true.
- complex conditional statements can group conditions using parentheses.

```bash
vim when_multiple.yml;
ansible-Playbook when_multiple.yml;
```

#### Combining Loops and Conditions

- Lops and conditionals can be combined.
- For intance, you can iterate through a list of dictionaries and apply the conditional statement only if a dictonary is found that matches the condition.

```bash
vim restart.yml;
ansible-playbook -v restart.yml;
ansible ansible2.example.com -m command -a "systemctl status crond";
ansible ansible2.example.com -m command -a "systemctl start crond";
ansible-playbook restart.yml;
```

### 6.5 Using Handlers

#### Understanding Handlers

- Handlers allow you to configure playbooks in a way that one task will only run if another task has been running successfully.
- In order to run the handler, a **notify** statement is used from the main task to trigger the handler.
- Handler typically are used to restart services or reboot hosts.
- Handlers are executed after running all tasks in a play.
- Handlers will only run if a task has changed something, so if an ok result instead of a changed rusult is reported, the hadler will not run.
- If one if the tasks fails, the handler will not run, but this may be overwritten using **force_handlers: True**
- One task  may trigger more than one handler.

```bash
vim handlers.yml;
ansible-playbook handlers.yml;
touch index.html;
ansible-playbook handlers.yml;

vim handlers.yml # delete force_handlers: True and save the file.
ansible-doc file;
ansible all -m file -a "path=/var/www/html/index.html state=absent"
ansible-playbook handlers.yml;
touch /tmp/nothing;
ansible all -m file -a "path=/var/www/html/index.html state=absent"
ansible-playbook handlers.yml;
```

### 6.6 Using Blocks

#### Understaing Ansible Blocks

- A block is a logical group of taks.
- It can be used to control how tasks are executed.
- One block can, for instance, be enabled using a single **when**
- Blocks can also be used in error condition handling.
  - Use **block** to define the main tasks to run.
  - Use **rescue** to define tasks that run if tasks defined in the **block** fail 
  - Use **always** to define tasks that will run, regardless of the success or failure of the **block** and **rescue** tasks.
- Notice that items cannot be used on blocks.

```bash
vim blocks.yml;
ansible-playbook blocks.yml;
vim blocks2.yml;
ansible-playbook blocks2.yml;
ansible ansible2.example.com -m file -a "path=/var/www/html/index.html state=touch"
ansible-playbook blocks2.yml;
```

### 6.7 Dealing with Failures

#### Understaing Failure Handling

- Ansible looks at the exit status of a task to determine whether it has failed.
- When any task fails, Ansible aborts the rest of the play on that host and continues with the next host.
- Different solutions can be used to change that behavior.
- Use **ignore_errors** in a task to ignore failures.
- Use **force_handlers** to force a handler that has been triggered to run, even if (another) task fails.

#### Defining Failure States

- The **failed_when** keyword can be used in a task to identify when a task has failed.
- The **fail** module can be used to print a message that informs why a task has failed.
- To use **failed_when** or **fail** the result of the command must be registered, and the registered variable ouput must be analyzed.
- When using the **fail** module, the failing task must have **ignore_errors** set to yes.

```bash
vim failure.yml;
ansible-playbook failure.yml;
vim failure2.yml;
ansible-playbook failure2.yml;
```

### 6.8 Managing Changed Status

#### Handling Changed Status

- Managing the changed status may be important, as handlers trigger on the changed status.
- The result of a command can be registered, and the registered variable can be scanned for specific text to determine that a change has occurred.
- This allow Ansible to reporta changed status, where it normally would not, thus allowing handlers to be triggered.
- Using **changed_when** is usable in two cases.
  - To allow handlers to run when a change would not rormally trigger.
  - To disable commands that run successful to reporta changed status.

```bash
vim changed.yml;
ansible-playbook changed.yml;
```

### Lesson 6 Lab: Using Task Control

- Write a playbook that will install and run the mariadb database service. It should install the mariadb-server as well as the python3-PyMySQL package, and it should only install on managed nodes that are using RHEL 8.0
- Only if the database service could be started successfully, the **mysql_user** module should be used to set the database root user password.

### Lesson 6 Lab Solution: Using Task Control

```bash
vim lab.yml;
ansible-playbook lab.yml;
```
