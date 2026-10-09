# TP1 Dockers

## Commands


### Database:
```
docker rm -f database
docker run -e POSTGRES_USER=usr -e POSTGRES_DB=db -e POSTGRES_PASSWORD=pwd --network app-network --name database -d raphaelaurent/database
```

### Backend:
```
docker rm -f backend
docker run --network app-network -p 8080:80 --name backend -d raphaelaurent/backend
```
Sans ports :
```
docker run --network app-network --name backend -d raphaelaurent/backend
```

### my-running-app :

```
docker rm -f my-running-app 
docker run -d -p 80:80 --network app-network --name my-running-app my-apache2
```


### Docker compose :
```
docker rm -f database backend my-running-app
docker compose up --build -d
```





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

### 3-3 Document your docker_container tasks configuration

```bash
- name: Run database 
  docker_container:
    name: database
    image: postgres:15
    env:
      POSTGRES_USER: "{{ db_user }}"
      POSTGRES_PASSWORD: "{{ db_password }}"
      POSTGRES_DB: "{{ db_name }}"
    networks: 
      - name: myNetwork
    volumes:
      - db_data:/var/lib/postgresql/data
  vars:
    ansible_python_interpreter: /opt/docker_venv/bin/python
```
```bash
- name: Run app 
  docker_container:
    name: backend
    image: raphaelaurent/backend
    env:
      DB_HOST: database
      DB_USER: "{{ db_user }}"
      DB_PASSWORD: "{{ db_password }}"
      DB_NAME: "{{ db_name }}"
    networks:
      - name: myNetwork
    ports:
      - "8080:80"
  vars:
    ansible_python_interpreter: /opt/docker_venv/bin/python
```
```bash
- name : Create app network
  docker_network:
    name: myNetwork
    state: present
  vars:
    ansible_python_interpreter: /opt/docker_venv/bin/python
```
```bash
- name : Run proxy
  docker_container:
    name: myProxy
    image: raphaelaurent/httpd
    networks:
      - name: myNetwork
    ports:
      - "80:80"
  vars:
    ansible_python_interpreter: /opt/docker_venv/bin/python
```