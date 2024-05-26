
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
- [Reddy](#reddy)
- [Docker installation ](#docker-installation-)
- [Inventory ](#inventory-)
  - [File structure](#file-structure)
    - [Host infomation](#host-infomation)
    - [Variables](#variables)
    - [Logical subgroups](#logical-subgroups)
- [Jinja 2 templates](#jinja-2-templates)
- [Semaphore UI ](#semaphore-ui-)
- [Using Semaphore to run a playbook](#using-semaphore-to-run-a-playbook)

## Reddy[![](https://raw.githubusercontent.com/aregtech/areg-sdk/master/docs/img/pin.svg)](#reddy)

Reddy is the system we will base our Ansible demonstration on. It consists of an [NGINX](https://nginx.org/en/) acting as a load balancer for two [Rust](https://www.rust-lang.org/)-based web servers. These servers store/retrieve a key in [Redis](https://redis.io/), and if it exists, return the associated value for that key. All of this is done using HTTP requests.

The system was set up using [Docker Compose](docker-compose.yaml), allowing us to conduct local testing and, ideally, add new containers for configuration with Ansible as needed. Each of the squares would represent a [Docker container](docker-containers). All containers belong to the same network, and the public key (id_rsa.pub) of the Ansible container is present in all containers to ensure seamless connections between them and enable task execution.

(INSERTAR IMAGEN DE LA TOPOLOGIA)

<div align="right">[ <a href="#table-of-contents">↑ Back to top ↑</a> ]</div>

## Docker installation [![](https://raw.githubusercontent.com/aregtech/areg-sdk/master/docs/img/pin.svg)](#docker-installation) 
In the root of the project, run docker compose (this will take a while).

```bash
$ docker-compose up -d
```

With this command, we are bringing up the aforementioned topology. However, before demonstrating it, before making any demonstration, we will provide an explanation of some concepts.

<div align="right">[ <a href="#table-of-contents">↑ Back to top ↑</a> ]</div>

## Inventory [![](https://raw.githubusercontent.com/aregtech/areg-sdk/master/docs/img/pin.svg)](#inventory) 
In this section, we will explain the structure and purpose of the Ansible inventory file used in the project. An Ansible inventory file is a YAML or INI file that defines the hosts and groups of hosts upon which Ansible commands, modules, and playbooks operate. The inventory file for this project can be found [here](inventory.yml).

### File structure

#### Host infomation
- The `hosts` subsection lists individual hosts with their connection information
- Each host is defined with a name and connection details
- The `ansible_host` variable specifies the address Ansible should use to connect to the host

#### Variables 
- Although not shown in this specific inventory file, the `vars` subsection can be used to define variables that apply to all hosts within the `all` group.

#### Logical subgroups
  The `children` subsection groups hosts into logical subgroups, making it easier to manage and apply tasks to related hosts collectively. There are 3 groups in the inventory file:

- **Webserver Group**
  - The `webserver` group contains hosts related to web servers.
  - Hosts in this group:
    - `web-server1`
    - `web-server2`

- **Rediserver Group**
  - The `rediserver` group contains hosts related to Redis servers.
  - Hosts in this group:
    - `redis-server1`

- **Loadbalancer Group**
  - The `loadbalancer` group contains hosts related to load balancers.
  - Hosts in this group:
    - `load-balancer`

<div align="right">[ <a href="#table-of-contents">↑ Back to top ↑</a> ]</div>

## Jinja 2 templates

Ansible uses Jinja 2 templating to enable dynamic expressions and access variables and facts (parametrization).  This avoids hardcoding values inside templates

Usually templates are stores in the ```templates``` module. For example, you can create a template for a configuration file, then deploy that configuration file to multiple environments and supply the correct data (IP Address, hostname, version) for each environment.

All templating happens on the ansible control node before the task is sent and executed on the target machine. This approach minimize the package requirements on the target (Jinja 2 is only required in the control node). It also limits the amount of data Anislbe passes to the target machine.
It also limits the amount of data Ansible passes to the target machine. Ansible parses templates on the control node and passes only the information needed for each task to the target machine, instead of passing all the data on the control node and parsing it on the target

<div align="right">[ <a href="#table-of-contents">↑ Back to top ↑</a> ]</div>

## Semaphore UI [![](https://raw.githubusercontent.com/aregtech/areg-sdk/master/docs/img/pin.svg)](#semaphore) 

[Semaphore UI](https://www.semui.co/) is an open-source project offering a responsive web UI for running Ansible playbooks. It simplifies workflow management by allowing users to efficiently execute tasks, organize playbooks, and manage environments, inventories, repositories, and access keys. With its mobile-friendly interface, Semaphore provides flexibility in task management, enabling users to schedule playbook runs, access detailed logs, delegate tasks, and receive notifications.

## Using Semaphore to run a playbook

Let's run a playbook using Semaphore. The service its listening in http://localhost:3000/. 


<div align="center">
    <img src="readme-utils/semaphore-images/login.png" alt="Semaphore Login">
</div>

Login with the following credentials:

```bash
username = admin
password = admin
```

Now create a new project called `ansible-demo` like the GIF shows:
<div align="center">
    <img src="readme-utils/semaphore-images/create-project.gif" alt="Create Project">
</div>

Create a repository using the link of the repo like the GIF shows:
<div align="center">
    <img src="readme-utils/semaphore-images/create-repo.gif" alt="Create Repo">
</div>

Create ssh keys using the private key of the ansible-container. To get it:
```bash
$ docker exec -it ansible-container /bin/bash
root@23fa96265dde:/ansible cat /root/.ssh/id_rsa
```
Now paste it like the GIF shows:
<div align="center">
    <img src="readme-utils/semaphore-images/create-key.gif" alt="Create Key">
</div>

Create the inventory like the GIF shows:
<div align="center">
    <img src="readme-utils/semaphore-images/create-inv.gif" alt="Create Inventory">
</div>

Create the environment like the GIF shows:
<div align="center">
    <img src="readme-utils/semaphore-images/create-env.gif" alt="Create Env">
</div>

Create the task like the GIF shows:
<div align="center">
    <img src="readme-utils/semaphore-images/create-task.gif" alt="Create task">
</div>

Now run the task. This will be the expected result:
<div align="center">
    <img src="readme-utils/semaphore-images/succesful-run.png" alt="Succesful task">
</div>

<div align="right">[ <a href="#table-of-contents">↑ Back to top ↑</a> ]</div>
