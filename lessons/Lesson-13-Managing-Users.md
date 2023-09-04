# Lesson 13 Managing Users

- [Lesson 13 Managing Users](#lesson-13-managing-users)
    - [13.1 Understanding Modules Related to User Management](#131-understanding-modules-related-to-user-management)
      - [Modules Related to User Management](#modules-related-to-user-management)
      - [Understanding authorized\_key](#understanding-authorized_key)
    - [13.2 Implementing a Playbook to Manage Users](#132-implementing-a-playbook-to-manage-users)
      - [Managing Users](#managing-users)
      - [Managing sudo](#managing-sudo)
    - [13.3 Managing Encrypted Passwords](#133-managing-encrypted-passwords)
      - [Understanding Encrypted Passwords](#understanding-encrypted-passwords)
      - [Understanding Encrypted Passwords](#understanding-encrypted-passwords-1)
      - [Creating Encrypted Passwords in Ansible.](#creating-encrypted-passwords-in-ansible)
    - [Lesson 13 Lab: Managing Users](#lesson-13-lab-managing-users)
    - [Lesson 13 Lab Solution: Managing Users](#lesson-13-lab-solution-managing-users)

### 13.1 Understanding Modules Related to User Management

#### Modules Related to User Management

- **user**: Managers Users and their common properties.
- **group**: Managers Groups and their common properties.
- **padm**: Condigures PAM.
- **authorized_key**: Copies SSH public keys from Ansible Control to the target user .ssh/authorized_keys
- **lineinfile**: Modfies configuration files based on regex.

#### Understanding authorized_key

- The **authorized_key** module can be used to copy a user public key to the ~/.ssh/authorized_key file
- This is useful to enable SSH access from remote machines, such as the Ansible control node.
- Note that this will only copy the public key, and does NOT generate anh SSH keys.
- To generate SSH keys for new users, use the **user** module and its option **generate_ssh_key**

### 13.2 Implementing a Playbook to Manage Users

#### Managing Users

- Some tasks have well defined modules, some tasks don't
- When no specific module exists, it comes down to your own creativity
- When creating users, you will at least want to use the **users** and **groups** modules, whereas **authorized_key** is useful as well.

```bash
vim setup_users.yml
cat vars/users
cat vars/groups
ansible-playbook setup_users.yml;
```

#### Managing sudo

- There is no Ansbile module for managing **sudo**
- Using templates and variables, it's not too difficult to set it up manually though.
- In the next demo, you will see how to use a playbook, together will a Jinja2 template and variables to set up a **sudo** configuration.

```bash
vim vars/defaults
vim sudoers.j2;
ansible-playbook setup_sudo.yml ;
```

### 13.3 Managing Encrypted Passwords

#### Understanding Encrypted Passwords

- Encrypted user passwords are stored in /etc/shadow
- A password strings looks like " ex: <encrypted password>"
- This string consistes of 3 parts.
  - The hashing algorithm that is used.
  - The random salt that was used to encrypt the password.
  - The encrypted has of the user password.

#### Understanding Encrypted Passwords

- To create an encrypted password, a random salt is used to ensure that wo users that have identical passwords would not have identical entries in /etc/shadow.
- This salt and the unencrypted password are combined and encrypted, which generates the encrypted has that is stored in /etc/shadow.

#### Creating Encrypted Passwords in Ansible.

- The Ansible user module does not generate encrypted passwords.
- To generate an encrypted password, an exteraal utility must be used to generate a cryptosting.
- Next, the cryptostring can be used in a variable to create the user password.
- For enhanced security, consider storing the password hash in a vault encrypted file.

```bash
ansible localhost -m debug -a "msg={{ 'password' | password_hash('sha512', 'mypassword')}}"

vim userpw.yml;
ansible-playbook userpw.yml;
ansible ansible2.example.com -m user -a "name=anil state=absent remove=yes"
ansible-playbook userpw.yml;
ssh anil@ansible2
```

### Lesson 13 Lab: Managing Users

- Continue on the setup_sudo configuration from the previous example, and  make it a complete solution to create users meeting the following requirements.
  - Groups in the vars/defaults file should be created if they don't exist yet.
  - Use a variable file to define users and the groups these users should be a member of, and create the following users.
    - bill and bob are members of the group admins.
    - bea and betty are members of the group developers.
    - brenda and benny are members of the group dbas.
- Do this in a way that SSH authrized kyes are set up automatically as well.

### Lesson 13 Lab Solution: Managing Users

- Modify the lesson13 YAML files to complete this lab.
