# Lesson 9 Using RHEL System Roles

- [Lesson 9 Using RHEL System Roles](#lesson-9-using-rhel-system-roles)
    - [9.1 Understanding RHEL System Roles](#91-understanding-rhel-system-roles)
      - [Current RHEL System Roles](#current-rhel-system-roles)
    - [9.2 Installing RHEL System Roles](#92-installing-rhel-system-roles)
    - [9.3 Using the RHEL SELinux System Role](#93-using-the-rhel-selinux-system-role)
      - [Using rhel-system-roles.selinux](#using-rhel-system-rolesselinux)
      - [Rebooting After Making Changes](#rebooting-after-making-changes)
      - [Setting SELinux Related Variables](#setting-selinux-related-variables)
    - [9.4 Using the RHEL TimeSync System Role](#94-using-the-rhel-timesync-system-role)
      - [Why Does it Make Sense?](#why-does-it-make-sense)
      - [Using rhel-system-role.timesync](#using-rhel-system-roletimesync)
    - [Lesson 9 Lab: Using RHEL System Roles](#lesson-9-lab-using-rhel-system-roles)
    - [Lesson 9 Lab Solution: Using RHEL System Roles](#lesson-9-lab-solution-using-rhel-system-roles)

### 9.1 Understanding RHEL System Roles

- RHEL system roles are provided to configure standard RHEL operations.
- RHEL system roles have been provided since RHEL 7.4, and can be used to configure RHEL 6.10 and later.
- Install the rhel-system-role package to use them.
- RHEL system roles are derived from the Ansible Linux System Roles project, which is available through Ansible Galaxy.

#### Current RHEL System Roles

- Curretnly, the following RHEL system roles are provided.
  - **rhel-system-roles.kdump** configures the **kdump** crash recovery service.
  - **rhel-system-roles.network** configures network interfaces.
  - **rhel-system-roles.selinux** manages all aspects of SELinux.
  - **rhel-system-roles.timesync** is used to set up Network Time Protocol or Precision Time Protocol.
  - **rhel-system-roles.posix** is used to configure a host a Postfix MTA.
  - **rhel-system-roles.firewall** configures a firewall.
  - **rhel-system-roles.tuned** configures the tuned service.
- Additional RHEL system roles are likely to be introduced.

### 9.2 Installing RHEL System Roles

- Use **yum install -y rhel-system-roles** to install them.
- The roles are installed to the /usr/share/ansible/roles directory, notice that the upstream linux-system-roles name is provided as a symbolic link to provide maximum compatibility.
- Documentation of Ansible system roles is available in /usr/share/doc/rhel-system-roles-<version>
- Look for example YAML files in the role directories.

```bash
su -
yum search rhel-system-roles;
yum install -y rhel-system-roles;
rpm -ql rhel-system-roles;
cd /usr/share/ansible/roles/
ls 
\ls -l;
cd rhel-system-roles.selinux/
ls;
tree
```

### 9.3 Using the RHEL SELinux System Role

#### Using rhel-system-roles.selinux

The RHEL system role for SELinux can do several things.
- Set enforcing or permissive mode.
- Set SELinux fir contexts.
- Run **restorecon**.
- Set booleans.
- Set SELinux User Mappings.

#### Rebooting After Making Changes

- In some cases, to apply SELinux changes (such as a switch between enabled and disabled mode), a reboot is required.
- The SELinux role doesn't reboot hosts because if should be up to the administrator to do that.
- The role will set the **selinux_reboot_required** variable to **true**, and fail if a reboot is required.
- This is used in a block /rescue structure, where the play is failing if the variable is not set to true.
- If the variable is set to true, the host is rebooted and the role is started again.
- See sample code in **example-selinux-playbook** in the roles documentation.

#### Setting SELinux Related Variables

- To Configure SELinux from the role, set at least the following variables.
  - **selinux_state**
  - **selinux_booleans**
  - **selinux_fcontexts**
  - **selinux_restore_dirs**
  - **selinux_ports**
- See the example plyabook for examples.

```bash
cd /usr/share/doc/rhel-system-roles/selinux
pwd
ls;
less README.md;
vim example-selinux-playbook.yml;
cd /usr/share/ansible/
ls;
cd roles/
ls;
cd rhel-system-roles.selinux/
tree;
vim tasks/main.yml;
```

### 9.4 Using the RHEL TimeSync System Role

#### Why Does it Make Sense?

- **rhel-system-roles.timesync** can be used to manage time synchronization
- As the **timesync** service is different between RHEL 6 and RHEL 7/8, it makes sense managing this funtionality using a rhel-system-role.

#### Using rhel-system-role.timesync

- The role itself is configured to work with different variables, of which **timesync_ntp_servers** is the most important one.
- Items in this variable are made up of different attributes, of which two are common.
  - **hostnames** shows that hostname of the time server.
  - **iburst** specifies that fast iburst sysnchronization should be used.
  - The **timezone** variable is also important, and sets the current timezone to be used.
  - A default playbook is available in /usr/share/doc/rhel-system-roles/timesync

```bash
cd /usr/share/doc/rhel-system-roles/timesync
ls;
vim example-timesync-playbook.yml;
vim example-timesync-pool-playbook.yml;
vim setuptime.yml;
vim setuptime-light.yml;
vim group_vars/servers/timesync.yml;
ansible-playbook setuptime-light.yml;
```

### Lesson 9 Lab: Using RHEL System Roles

- Use a RHEL System Role to set up network cards in managed machines. You can use a simple configuration where the interface is set to DHCP.

### Lesson 9 Lab Solution: Using RHEL System Roles

```bash
cp /usr/share/doc/rhel-system-roles/network/example-eth-simple-auto-playbook.yml setup-network.yml # Copy only if it is doesn't exist in the project directory.

vim setup-network.yml; # Change the mac address.
ansible ansible2.example.com -a 'ip a' # To get the mac address.
ansible-playbook setup-network.yml;
ansible ansible2.example.com -a 'ip a'
ansible ansible2.example.com -a 'nmcle con show'
```
