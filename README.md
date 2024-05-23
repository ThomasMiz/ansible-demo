
<h1 align="center" style="display: block; font-size: 2.5em; font-weight: bold; margin-block-start: 1em; margin-block-end: 1em;">
<a name="logo" href="https://docs.ansible.com/ansible/latest/index.html"><img align="center" src="https://logos-download.com/wp-content/uploads/2016/10/Ansible_logo.png" alt="ANSIBLE" style="width:50%;height:100%"/></a>
  <br /><br /><strong>ANSIBLE DEMO</strong>
</h1>

<div align="center">
  <img src="https://img.shields.io/badge/Ansible-000000?style=for-the-badge&logo=ansible&logoColor=white" alt="ansible">
  <img src="https://img.shields.io/badge/redis-%23DD0031.svg?&style=for-the-badge&logo=redis&logoColor=white" alt="redis">
  <img src="https://img.shields.io/badge/Docker-2CA5E0?style=for-the-badge&logo=docker&logoColor=white" alt="docker">
  <img src="https://img.shields.io/badge/Rust-000000?style=for-the-badge&logo=rust&logoColor=white" alt="rust">
  <img src="https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white" alt="nginx">
</div>

---

## Introduction

[Ansible](https://docs.ansible.com/ansible/latest/index.html) is an open-source IT automation tool that simplifies configuration management, application deployment, and infrastructure orchestration. It uses a simple declarative language based on YAML to describe the desired system configuration and automate complex tasks efficiently. Ansible is highly scalable and can manage everything from small single-server setups to large enterprise environments with thousands of nodes. Being agentless (unlike [Puppet](https://www.puppet.com/)), it doesn't require additional software on the target nodes, making deployment easy and reducing operational complexity.

## Table of contents[![](https://raw.githubusercontent.com/aregtech/areg-sdk/master/docs/img/pin.svg)](#table-of-contents) 
- [Introduction](#introduction)
- [Table of contents](#table-of-contents)
- [Reddy ](#reddy-)
- [Docker installation ](#docker-installation-)

## Reddy [![](https://raw.githubusercontent.com/aregtech/areg-sdk/master/docs/img/pin.svg)](#reddy)

Reddy is the system we will base our Ansible demonstration on. It consists of an [NGINX](https://nginx.org/en/) acting as a load balancer for two [Rust](https://www.rust-lang.org/)-based web servers. These servers store/retrieve a key in [Redis](https://redis.io/), and if it exists, return the associated value for that key. All of this is done using HTTP requests.

The system was set up using [Docker Compose](docker-compose.yaml), allowing us to conduct local testing and, ideally, add new containers for configuration with Ansible as needed. Each of the squares would represent a [Docker container](docker-containers). All containers belong to the same network, and the public key (id_rsa.pub) of the Ansible container is present in all containers to ensure seamless connections between them and enable task execution.

(INSERTAR IMAGEN DE LA TOPOLOGIA)

<div align="right">[ <a href="#table-of-contents">↑ Back to top ↑</a> ]</div>

## Docker installation [![](https://raw.githubusercontent.com/aregtech/areg-sdk/master/docs/img/pin.svg)](#docker-installation) 
In the root of the project, run docker compose (this will take a while).

```bash
$ docker-compose up -d
```

Connect to the ansible-container
```bash
$ docker exec -it ansible-container /bin/bash
```

Now build reddy executable file with the following command:
```bash
root@f1fe1a8fdfd7:/ansible$ cd reddy && cargo build && cd ..
```

Run the playbook:
```bash
root@f1fe1a8fdfd7:/ansible$ ansible-playbook playbook.yml
```

<div align="right">[ <a href="#table-of-contents">↑ Back to top ↑</a> ]</div>

## Using Semaphore (GUI)

Semaphore is available at 
```bash
http://localhost:3000/
username = admin
password = changeme
```

### 1) Add SSH Key

In ansible-container run the following command to get the key
```bash
root@23fa96265dde:/ansible# cat /root/.ssh/id_rsa
```

In semaphore go to KeyStore
Add new key :
    -Type : SSH-Key 
    -Username: root 
    -Passphrase : None
    - Copy paste private key


### 2) Create Inventory

Go to Inventory section and create Inventory :
    - UserCrendtials : Select the Key imported at previous step
    - type : Static yaml -> copy paste content of static.yml

### 3) Create Repo

Go to Repository section and create repo
    - Branche : main
    - Access-Key : None 

### 4) Create Task 

Go to Tasks section and create task , specify the inventory , repo created previsouly and add the playbook you want to run 

### 5) Execute Task 


