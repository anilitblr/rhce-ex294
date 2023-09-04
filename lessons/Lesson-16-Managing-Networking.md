# Lesson 16 Managing Networking

- [Lesson 16 Managing Networking](#lesson-16-managing-networking)
    - [16.1 Using Network Roles for Network Management](#161-using-network-roles-for-network-management)
      - [Using the Network System Role](#using-the-network-system-role)
      - [Setting Variables in the RHEL 8 System Roles](#setting-variables-in-the-rhel-8-system-roles)
    - [16.2 Understanding Modules for Network Management](#162-understanding-modules-for-network-management)
      - [Understanding Modules](#understanding-modules)
    - [16.3 Using Ansible to Manage IPv6](#163-using-ansible-to-manage-ipv6)
    - [Lesson 16 Lab: Managing Networking](#lesson-16-lab-managing-networking)
    - [Lesson 16 Lab Solution: Managing Networking](#lesson-16-lab-solution-managing-networking)

### 16.1 Using Network Roles for Network Management

#### Using the Network System Role

- RHEL 8 includes a RHEL system role to manage networking.
- Use **ansible-galaxy list** for an overview of all RHEL 8 system roles.
- See contents of the roles in /usr/share/ansbile/roles

#### Setting Variables in the RHEL 8 System Roles

- Two variables are used to configure the network role.
  - **network_provider**: typically set to **nm**
  - **network_connections**: specifies details about the network connection.
- Defaults for these variables are set in /usr/share/ansible/roles/rhel-system-roles.network

```bash
cd /usr/share/ansible/roles/
ls;
cd rhel-system-roles.network/
ls;
cd tasks/
vim main.yml;
cd /usr/share/doc/rhel-system-roles/network/
ls;
vim example-bond-with-vlan-playbook.yml;
cd lesson16/
vim setupnw.yml;
vim networkvars.yml;
ansible-playbook setupnw.yml;
```

### 16.2 Understanding Modules for Network Management

#### Understanding Modules

- Different modules are available for managing network settings.
- **nmcli** is used to mange many parameters for network devices or connections.
- **hostname** can be used to set the name of a managed host
- **firewalld** can be used to mange Firewalld rules.

```bash
ansible all -m setup -a 'gather-subset=network filter=ansible_interfaces'
ansible all -m setup -a 'gather-subset=network filter=ansible_ens224'

```bash
vim setup_nic.yml;
```

```yaml
---
- name: setup a NIC
  hosts: ansible2.example.com
  tasks:
  - name: configure ens224
    nmcli: 
      conn_name: new-conn
      ifname: ens224
      type: ethernet
      ip4: 10.0.0.10/24
      state: present
  - name: set hostname
    hostname:
      name: ansible2.example.com
  - name: move ens224 to internal zone
    firealled:
      zone: internal
      interface: ens224
      permanent: yes
      state: enabled
  - name: enable http in firewall internal zone
    firewalld:
      zone: internal
      service: http
      permanent: yes
      state: enabled
```

```bash
ansible-playbook setup_nic.yml;
```

### 16.3 Using Ansible to Manage IPv6

```bash
vim ipv6.yml;
```
```yaml
---
- name: setup a NIC
  hosts: ansible2.example.com
  tasks:
  - name: configure ens224
    nmcli:
      conn_name: ipv6-conn
      ifname: ens224
      type: ethernet
      ip4: 192.168.1.10/24
      ip6: fc00::202/64
      gw4: 192.168.1.1
      gw6: fc00::1
      dns4: 8.8.8.8
      dns6: 2001:4860:4860:8888
      state: present
```

```bash
ansible-doc nmcli;
ansible-playbook ipv6.yml;

```

### Lesson 16 Lab: Managing Networking

- Write a playbook that verifies that two network interfaces are available.
- If a second interface is available, the playbook should set it up with network configuration and assign the IP address 192.168.1.30/24 on the interface.

### Lesson 16 Lab Solution: Managing Networking

- vim lab.yml

```yaml
---
- name: configure NIC
  hosts: all
  roles:
    - rhel-system-roles.network
```

```bash
cat host_vars/ansible2.example.com/network.yml
ansible-playbook lab.yml;
ansible all -m setup -a 'filter=ansible_ens224'
```
