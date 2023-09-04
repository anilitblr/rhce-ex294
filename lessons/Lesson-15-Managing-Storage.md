# Lesson 15 Managing Storage

- [Lesson 15 Managing Storage](#lesson-15-managing-storage)
    - [15.1 Understanding Modules for Managing Storage](#151-understanding-modules-for-managing-storage)
      - [Understanding Modules for Managing Storage](#understanding-modules-for-managing-storage)
    - [15.2 Implementing a Playbook to Manage Storage](#152-implementing-a-playbook-to-manage-storage)
    - [Lesson 15 Lab: Managing Storage](#lesson-15-lab-managing-storage)
    - [Lesson 15 Lab Solution: Managing Storage](#lesson-15-lab-solution-managing-storage)

### 15.1 Understanding Modules for Managing Storage

#### Understanding Modules for Managing Storage

- **parted**: runs the parted utility
- **lvg**: create LVM volume group
- **lvol**: creates LVM logical volumes
- **filesystem**: managed filesystems
- **mount**: manages mounts
- **vdo**: manages VDO storage

### 15.2 Implementing a Playbook to Manage Storage

```bash
vim setup-storage.yml;
cat vars/storage.yml;
ansible-playbook setup-storage.yml;
```

### Lesson 15 Lab: Managing Storage

- Write a playbook that detects all storage devices on managed ststems.

### Lesson 15 Lab Solution: Managing Storage

- vi lab.yml

```yaml
---
- name: Gather facts for disk devices
  gather_facts: false
  hosts: all
  vars:
    disks: []

  tasks:
    - name: setup
      setup:
      register: factone
    - set_fact:
        all_devices: "{{ factone.ansible_facts.ansible_devices }}"
    
    - name: Returen data fro parent block devices
      set_fact:
        disks: "{{ disks + [ item.key ] }}"
      with_dict: "{{ all_devices }}"
      when: item.value.host
      
    - name: diag
      debug:
        msg: " {{ disks }}"
    
    - name: pick 2nd disk
      debug:
        msg: "{{ disks[1] }}"
```

```bash
ansible-playbook lab.yml;
```
