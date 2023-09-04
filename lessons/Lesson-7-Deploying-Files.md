# Lesson 7 Deploying Files

- [Lesson 7 Deploying Files](#lesson-7-deploying-files)
    - [7.1 Using Modules to Manipulate Files](#71-using-modules-to-manipulate-files)
      - [Common File Modules](#common-file-modules)
    - [7.2 Managing SELinux File Context](#72-managing-selinux-file-context)
      - [Managing SELinux Context](#managing-selinux-context)
    - [7.3 Using Jinja2 Templates](#73-using-jinja2-templates)
      - [Understaning Jinja2 Templates](#understaning-jinja2-templates)
      - [Avoiding Confusion When Using templates](#avoiding-confusion-when-using-templates)
    - [7.4 Using Control Structures in Jinja2](#74-using-control-structures-in-jinja2)
      - [Using Control Structures in Templates](#using-control-structures-in-templates)
    - [Lesson 7 Lab: Deploying Files with Templates](#lesson-7-lab-deploying-files-with-templates)
    - [Lesson 7 Lab Solution: Deploying Files with Templates](#lesson-7-lab-solution-deploying-files-with-templates)

### 7.1 Using Modules to Manipulate Files

#### Common File Modules

Different Modules are available for managing files.

- **lineinfile**: ensures that a line is in a file, useful for changing a single line in a file.
- **blockinfile**: manipulates multi-line blocks of text in files.
- **copy**: copies a file from a local or remote machine to a location on a managed host.
- **fetch**: used to fetch a file from a remote a=machine and store it on the management node.
- **file**: sets attributes to files, and can also create and remove files, symbolic links and more. 

```bash
vim  file.yml;
ansible-doc file; # Search for state
ansible-playbook file.yml;

vim copy.yml;
ansible-doc stat;
ansible-doc fetch;
ansible-playbook copy.yml;
```

### 7.2 Managing SELinux File Context

#### Managing SELinux Context

- **file**: sets attributes to files, including SELinux context, and can also crete and remove files, symbolic links and more.
- **sefcontext**: manages SELinux file context in the SELinux Policy (but not on files)
- Notice that file sets SELinux context directly on the file (like the **chcon** command), and not in the policy.

```bash
vim selinux.yml
ansible-playbook selinux.yml;
ansible all -a "ls -lZ /tmp/removeme";
```

### 7.3 Using Jinja2 Templates

#### Understaning Jinja2 Templates

- **lineinfile** and **blockinfile** can be used to apply simple modifications to files.
- For more advanced modifications, use Jinja2 templates.
- While using templates, the target files are automatically customized using variables and facts.
- In a Jinja2 template, you will find multiple elements.
  - data
  - variables
  - expressions
  - control Structures
- The variables in the template are replaced with their values when the Jinja2 template is rendered to the target file on the managed host.
- If using variables, they can be specifed using the **vars** section of the playbook.
- Alternatively, Ansible facts can be used as variables.

#### Avoiding Confusion When Using templates

- To prevent administrators from overwriting files that are manged by Ansible, set the **ansible_managed** string
  - First, in **ansible.cfg** set **ansible_managed = Ansible managed**
  - On top of the Jinja2 template, set the {{ ansible_managed }} variable

```bash
vim vsftpd-template.yml;
vim templates/vsftpd.j2
ansible-playbook vsftpd-template.yml;
ansible all -a "cat /etc/vsftpd/vsftpd.conf"
```

### 7.4 Using Control Structures in Jinja2

#### Using Control Structures in Templates

- In Jinja2 templates, control structures can be used to organize the template in an optinal way.
- The **for** statement can be used to iterate through a variable and use all values in the variable.
- The **if** statement can be used to have the template work with a variable if another variable is defined.

```bash
vim hostsfile.yml;
vim templates/hosts.j2
ansible-playbook hostsfile.yml;
ansible all -a "cat /etc/hosts"
```

### Lesson 7 Lab: Deploying Files with Templates

- To configure Anonymouse FTP upload, you will need to make sure that the following is accomplished.
  - vsftpd.conf is modified to allow anonymouse FTP access and uploads.
  - The directory /var/ftp/pub is configured with the appropriate permissions.
  - The directory /var/ftp/pub is configured with the appropriate SELinux context lable.
  - The SELinux boolean ftpd_anon_write is set to on.
- Create a playbook taht ensures that the vsftpd service is installed, enabled, the firewall is opened, and the above requirements are met. Define variables in the playbook to set vsftpd.conf parameters, and use these in a template.
- At the end of the playbook, verify connectivity, uploading the /etc/hosts file from localhost.

### Lesson 7 Lab Solution: Deploying Files with Templates

```bash
vim lab7.yml ;
ansible-playbook lab7.yml;
vim lab-test.yml;
ansible-playbook lab-test.yml;
ansible ansible2.example.com -a "firewall-cmd --list-all";
```
