# TP1 Dockers


# Questions

## TP1
#### 1-1 For which reason is it better to run the container with a flag -e to give the environment variables rather than put them directly in the Dockerfile?
The flag -e make it more secure as it make us avoid writting the values directly in the Dockerfile.

#### 1-2 Why do we need a volume to be attached to our postgres container?
We need a volume to persist data beyond the container's lifetime. With a volume, the data can survive even after the container is deleted.

#### 1-3 Document your database container essentials: commands and Dockerfile.
Dockerfile:
```bash
FROM postgres:17.2-alpine
COPY SQL/ /docker-entrypoint-initdb.d/
```
```bash
Build and run commands of the database container:
docker build -t raphaelaurent/database .
docker run -p 8888:5000 -e POSTGRES_USER=usr -e POSTGRES_DB=db -e POSTGRES_PASSWORD=pwd --network app-network -v /my/own/datadir:/var/lib/postgresql/data --name myDatabase -d raphaelaurent/database
```

#### 1-4 Why do we need a multistage build? And explain each step of this dockerfile.
Multistage allow us to use separate stages: one image to compile (JDK and Maven) and one image to execute (JRE). With multistage, we don’t need to have the build tools in the final image, making it lighter.
```
Dockerfile:

- FROM eclipse-temurin:21-jdk-alpine AS myapp-build : import of the JDK image
-	ENV MYAPP_HOME=/opt/myapp : path variable
-	WORKDIR $MYAPP_HOME : definition of the work directory
-	RUN apk add --no-cache maven : downloading Maven
-	COPY pom.xml . // COPY src ./src : copying the project in the image
-	RUN mvn package -DskipTests : compiling and creating the .jar
-	FROM eclipse-temurin:21-jre-alpine : creating a new image from a jre base
-	ENV MYAPP_HOME=/opt/myapp : path variable
-	WORKDIR $MYAPP_HOME : definition of the work directory
-	COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar : copy the .jar from the first stage
-	ENTRYPOINT ["java", "-jar", "myapp.jar"] : launch the app at the start
```

#### 1-5 Why do we need a reverse proxy?
A reverse proxy is a server placed in front of web server which transmit the client’s requests to those servers. We need it to provide a single-entry point, hide the backend and distribute the work between the servers.

#### 1-6 Why is docker-compose so important?
Docker-compose allow us to run a multi-container application. Instead of having to type multiple dockers build and run command for each container, we can use docker compose up to build and start the application.

#### 1-7 Document docker-compose most important commands. 
-	Docker compose up: create and start the services
-	Docker compose up --build: rebuild the images
-	Docker compose up --build -d: start the app in the background
-	Docker compose down: stop and delete the containers
-	Docker compose ps: list the services and informations about them
-	Docker compose logs <service>: display the <service>’s logs
-	Docker compose build: build the images without starting

#### 1-8 Document your docker-compose file.
```yml
services:
    backend:
        build: ./Backend
        networks:
        - app-network
        depends_on:
        - database

    database:
        build: ./Database
        environment: 
          POSTGRES_USER: ${POSTGRES_USER}
          POSTGRES_DB: ${POSTGRES_DB}
          POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
        networks:
        - app-network
        volumes:
        - db-data:/var/lib/postgresql/data

    httpd:
        build: ./HTTP_server
        ports:
        - "80:80"
        networks:
        - app-network
        depends_on:
        - backend

networks:
  app-network:
    driver: bridge

volumes:
  db-data: 
    driver: local
```

#### 1-9

##### Database:
```
docker rm -f database
docker run -e POSTGRES_USER=usr -e POSTGRES_DB=db -e POSTGRES_PASSWORD=pwd --network app-network --name database -d raphaelaurent/database
```

##### Backend:
```
docker rm -f backend
docker run --network app-network -p 8080:80 --name backend -d raphaelaurent/backend
```
Sans ports :
```
docker run --network app-network --name backend -d raphaelaurent/backend
```

##### my-running-app :

```
docker rm -f my-running-app 
docker run -d -p 80:80 --network app-network --name my-running-app my-apache2
```


##### Docker compose :
```
docker rm -f database backend my-running-app
docker compose up --build -d
```






## TP2 :
#### 2-1 What are testcontainers ?
Testcontainers is a library that gives acess to disposable docker containers to run services.

#### 2-2 For what purpose do we need to use secured variables ?
The secured variables are used to registered encrypted sensible information. For example, we can use them to register identifiers, and avoid putting them in the source code.

#### 2-3 Why did we put needs: build-and-test-backend on this job ?
This line assures that the job build-and-push-docker-image only run if the job test-backend is successfull. This insure that the image published is not bugged.

#### 2-4 For what purpose do we need to push docker images ?
We push docker images to store them. We can then share and download them on other machines, allowing us to deploy an app on other devices.

## TP3 :
#### 3-1 Document your inventory and base commands

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

#### 3-2 Document your playbook

playbook.yml :
```bash
- hosts: all
  gather_facts: true
  become: true

  roles:
    - docker #calling the role docker
```

#### 3-3 Document your docker_container tasks configuration

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

#### 3-4 Is it really safe to deploy automatically every new image on the hub? What can I do to make it more secure?
No it is not completely safe as an image can be bugged or can contain malicious code. To make it more secure, we can test the images before publishing the image to detect bugs or malicious codes.