

# Questions

### 3-1 Document your inventory and base commands

Inventory :
```bash
all:
  vars:
    ansible_user: admin
    ansible_ssh_private_key_file: ~/id_rsa
  children:
    prod:
      hosts: raphael.laurent.takima.school
```

Base commands :

```bash
ansible all -i inventories/setup.yml -m ping
```
```bash
ansible all -i inventories/setup.yml -m setup -a filter=ansible_distribution*"

```
```bash
ansible all -i inventories/setup.yml -m apt -a "name=apache2 state=absent" --become
```

### 3-2 Document your playbook

playbook.yml :
```bash
- hosts: all
  gather_facts: true
  become: true

  roles:
    - docker #calling the role docker
```