
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
- [Topology](#topology)
- [Docker installation](#docker-installation)
  - [Ansible Container (ansible-container)](#ansible-container-ansible-container)
  - [Semaphore Container (semaphore-container)](#semaphore-container-semaphore-container)
  - [Rust Container (rust-container)](#rust-container-rust-container)
  - [The remaining containers](#the-remaining-containers)
- [Inventory](#inventory)
  - [File structure](#file-structure)
    - [Host infomation](#host-infomation)
    - [Variables](#variables)
    - [Logical subgroups](#logical-subgroups)
- [Playbook](#playbook)
  - [How does a module run in the target node](#how-does-a-module-run-in-the-target-node)
- [Jinja 2 templates](#jinja-2-templates)
  - [Templating process](#templating-process)
  - [Usage of Jinja2 templates in this project](#usage-of-jinja2-templates-in-this-project)
- [Semaphore UI](#semaphore-ui)
  - [Using Semaphore to run a playbook](#using-semaphore-to-run-a-playbook)
- [Running Ansible through the terminal](#running-ansible-through-the-terminal)

## Topology[![](https://raw.githubusercontent.com/aregtech/areg-sdk/master/docs/img/pin.svg)](#topology)

The toppology consists of an [NGINX](https://nginx.org/en/) acting as a load balancer for two [Rust](https://www.rust-lang.org/)-based web servers. These servers store/retrieve a key in [Redis](https://redis.io/), and if it exists, return the associated value for that key. All of this is done using HTTP requests.

The system was set up using [Docker Compose](docker-compose.yaml), allowing us to conduct local testing and, ideally, add new containers for configuration with Ansible as needed. Each of the squares would represent a [Docker container](docker-containers). All containers belong to the same network, and the public key (`id_rsa.pub`) of the Ansible container is present in the `authorized_keys` file in all containers to ensure seamless connections between them and enable task execution via SSH.

<div align="center">
    <img src="readme-utils/topology.png" alt="Semaphore Login" width="738">
</div>

<div align="right">[ <a href="#table-of-contents">↑ Back to top ↑</a> ]</div>

## Docker installation[![](https://raw.githubusercontent.com/aregtech/areg-sdk/master/docs/img/pin.svg)](#docker-installation) 
In the root of the project, run docker compose. This will take a while, ~6 minutes.

```bash
$ docker-compose up -d
```
With this command, we are bringing up the aforementioned topology.

### Ansible Container (ansible-container)
The [Ansible container's Dockerfile](docker-containers/ansible-container/Dockerfile) only configures Ansible and generates SSH keys. To simplify the demonstration and emphasize Ansible's functionality, a volume has been mounted on all containers at `/root/.ssh`, containing the generated private/public key pair (`id_rsa` and `id_rsa.pub`) along with an authorized_keys file containing the corresponding public key. This setup enables all target containers to allow access to Ansible nodes via SSH. Additionally, both Ansible containers (`ansible-container` and `semaphore-container`) utilize the same keys to communicate with these nodes.

### Semaphore Container (semaphore-container)
The Semaphore container is responsible for bringing up the necessary user interface to use the application. This container relies on another PostgreSQL container to persist certain information (such as playbook runs, statistics, configurations, etc).

### Rust Container (rust-container)
In this topology, we can think of the [Rust container](docker-containers/rust-container/Dockerfile) as a pipeline in GitHub Actions or Jenkins. Essentially, it fetches from the [Reddy](https://github.com/ThomasMiz/reddy) repository, compiles the Rust project (required for the topology's webservers), and, via a volume, provides the executable file named `reddy` to the ansible-container and semaphore-container for use in the playbooks and be able to deploy it.

For instance, if we had opted for AWS instead of Docker, the GitHub Actions pipeline would have employed the checkout action, compiled the project using a Rust action, and delivered the `reddy` executable to the VMs with Ansible using SSH with a specific user.

### The remaining containers
The remaining containers are [Ubuntu containers](docker-containers/ubuntu-container/Dockerfile) with OpenSSH server installed so that Ansible can connect to them. They will act as the target nodes in our inventory.

 With the explanation of the containers, now we will provide an explanation of some Ansible concepts.

<div align="right">[ <a href="#table-of-contents">↑ Back to top ↑</a> ]</div>

## Inventory[![](https://raw.githubusercontent.com/aregtech/areg-sdk/master/docs/img/pin.svg)](#inventory) 
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

- **Rediservers Group**
  - The `rediservers` group contains hosts related to Redis servers.
  - Hosts in this group:
    - `redis-server1`
    - `redis-server2`

- **Loadbalancer Group**
  - The `loadbalancer` group contains hosts related to load balancers.
  - Hosts in this group:
    - `load-balancer`

<div align="right">[ <a href="#table-of-contents">↑ Back to top ↑</a> ]</div>

## Playbook[![](https://raw.githubusercontent.com/aregtech/areg-sdk/master/docs/img/pin.svg)](#playbook) 

Playbooks define automation routines in a declarative manner. They specify the desired state that the target needs to achieve for a given task and are written in YAML format.

A playbook consists of one or more plays listed in order. Each play contributes to the overall goal of the playbook by executing one or more tasks. Each task invokes an Ansible module, which is a functional unit within Ansible used to enact changes in the system. Essentially, modules enforce the desired state on a remote node.

At a minimum, each play defines:

- The managed nodes to be targeted using a pattern.
- One task to be executed.

### How does a module run in the target node
<div align="center">
    <img src="readme-utils/playbooks/module.png" alt="module" width="738">
</div>

The control node establishes an SSH connection with a remote node. Following this, the control node transfers the module to the remote node. To execute the module, the control node dispatches specific arguments to the copied module. Subsequently, the module operates on the remote node, and the resulting output is captured and returned to the control node. Executing any playbook or ad-hoc Ansible command will display this output.

While not all modules follow this exact process, it provides a helpful insight into the underlying operations.

<div align="right">[ <a href="#table-of-contents">↑ Back to top ↑</a> ]</div>

## Jinja 2 templates[![](https://raw.githubusercontent.com/aregtech/areg-sdk/master/docs/img/pin.svg)](#inventory) 

Ansible uses Jinja 2 templating to enable dynamic expressions and access variables and facts (parametrization).  This avoids hardcoding values inside templates

Usually templates are stores in the ```templates``` module. For example, you can create a template for a configuration file, then deploy that configuration file to multiple environments and supply the correct data (IP Address, hostname, version) for each environment.

All templating happens on the ansible control node before the task is sent and executed on the target machine. This approach minimize the package requirements on the target (Jinja 2 is only required in the control node). It also limits the amount of data Ansible passes to the target machine. Ansible parses templates on the control node and passes only the information needed for each task to the target machine, instead of passing all the data on the control node and parsing it on the target.

### Templating process

The image illustrates the process of templating in Ansible using Jinja2:

<div align="center">
    <img src="readme-utils/jinja2/jinja-template.png" alt="Semaphore Login" width="738">
</div>

1. **Template**: The initial template file, typically stored in the `templates` directory.
2. **J2 Engine**: The Jinja2 templating engine processes the template file on the Ansible control node.
3. **Rendered Template**: The processed template with all variables and expressions evaluated and replaced.
4. **Copy**: The rendered template is then copied to the target nodes (e.g., Node 1 and Node 2) for execution.

### Usage of Jinja2 templates in this project

Below is an example of how Jinja2 is used to dynamically add web servers in the NGINX configuration file. This example demonstrates how you can use a Jinja2 template to iterate over a group of web servers and configure them for load balancing. This aproach allows to add new web servers without changing the templates, it is not necessary to know the server's IPs, nor how many of them are being used:

```jinja
upstream app_servers {
    {% for server in groups['webserver'] %}
    server {{ hostvars[server]['ansible_host'] }}:8080 weight=1;
    {% endfor %}
}

server {
    listen 80;

    location / {
        proxy_pass http://app_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        add_header X-Backend-Server $upstream_addr;
    }
}
```

In this template:

- In the `upstream app_servers` block servers are dynamically added from the `webserver` group.
- The `for` loop iterates over each server in the `webserver` group.
- `hostvars[server]['ansible_host']` is used to get the IP address of each server.
- The resulting configuration sets up load balancing for the NGINX server with the dynamically generated list of backend servers.

Jinja2 template is also used to configure the `REDIS_HOST` environment variable for a web server. This template is part of a larger Ansible task that sets up the environment for a web server:

```jinja
REDIS_HOST=redis://{{redis_hostname}}:6379
```

The Ansible playbook has the following task: 

```yml
- name: Create .env file
  ansible.builtin.template:
    src: ./templates/env.j2
    dest: /etc/reddy/.env
  vars:
    redis_hostname: redis-server1
```

When the task runs, Ansible processes the `env.j2` template and replaces the `{{ redis_hostname }}` placeholder with `redis-server1`.

<div align="right">[ <a href="#table-of-contents">↑ Back to top ↑</a> ]</div>

## Semaphore UI[![](https://raw.githubusercontent.com/aregtech/areg-sdk/master/docs/img/pin.svg)](#semaphore) 

[Semaphore UI](https://www.semui.co/) is an open-source project offering a responsive web UI for running Ansible playbooks. It simplifies workflow management by allowing users to efficiently execute tasks, organize playbooks, and manage environments, inventories, repositories, and access keys. With its mobile-friendly interface, Semaphore provides flexibility in task management, enabling users to schedule playbook runs, access detailed logs, delegate tasks, and receive notifications.

### Using Semaphore to run a playbook

Let's run a playbook using Semaphore. The service its listening in http://localhost:3000/. 


<div align="center">
    <img src="readme-utils/semaphore-images/login.png" alt="Semaphore Login" width="738">
</div>

Login with the following credentials:

```bash
username = admin
password = admin
```

Now create a new project called `ansible-demo` as shown:
<div align="center">
    <img src="readme-utils/semaphore-images/create-project.gif" alt="Create Project">
</div>

- Click on _New Project_.
- Enter `ansible-demo` as the project name.
- Leave the other fields empty and click _Create_.

Then create a repository using the link of the git repository:
<div align="center">
    <img src="readme-utils/semaphore-images/create-repo.gif" alt="Create Repo">
</div>

- On the navigation sidebar to the left, go to _Repositories_.
- Click the _New Repository_ button on the top-right of the screen.
- Enter `ansible-demo` as the name of the repositroy and enter the URL of this git repository, https://github.com/ThomasMiz/ansible-demo.
- On _Branch_, specify the _main_ branch.
- On _Access Key_, select _None_.
- Click on the _Create_ button.

Next, we'll create the SSH keys using the private key of the ansible-container. To get this key, we need to open a terminal within the container, then we can find the key at `/root/.ssh/id_rsa`:
```bash
$ docker exec -it ansible-container /bin/bash
root@23fa96265dde:/ansible cat /root/.ssh/id_rsa
```

And now we can enter this key into Semaphore:
<div align="center">
    <img src="readme-utils/semaphore-images/create-key.gif" alt="Create Key">
</div>

- Now on the navigation sidebar, go to _Key Store_.
- Create a new key with the button on the top-right.
- Name the key `ssh-key`.
- On _Key Type_, select _SSH Key_. This will make more options appear.
- Enter `root` as the username, but leave the passphrase blank.
- Copy and paste the container's key into the _Private Key_ textbox.

With that done, let's set up the Ansible inventory for Semaphore:
<div align="center">
    <img src="readme-utils/semaphore-images/create-inv.gif" alt="Create Inventory">
</div>

- On the navigation sidebar, go to _Inventory_.
- Create a new inventory with the button on the top-right.
- Name the new inventory `inventory`.
- Set the user credentials to `ssh-key`.
- Leave _Sudo Credentials_ blank.
- On _Type_, select _File_. This will make more options appear.
- On _Repository_, select `ansible-demo`.
- On _Path to Inventory file_, type `inventory.yml`.

Next up, we'll configure a Semaphore environment:
<div align="center">
    <img src="readme-utils/semaphore-images/create-env.gif" alt="Create Env">
</div>

- On the navigation sidebar, go to _Environment_.
- Create a new environment with the button on the top-right.
- Enter `TESTING` as the environment name.
- Leave the other textboxes unmodified, as our playbooks don't require any additional environment variables.
- Save the environment.

And create a task for running our playbook:
<div align="center">
    <img src="readme-utils/semaphore-images/create-task.gif" alt="Create task">
</div>

- On then navigation sidebar, go to _Task Templates_.
- Create a new task template with the button on the top-right.
- Enter "Run Playbook" as the name of the task
- Enter `playbook.yml` as the _Playbook Filename_.
- Select `ansible-demo` as the repository.
- Select `TESTING` as the environment.
- Leave all fields blank and press _Create_.

Finally, we can run the task! In _Task Templates_ we will now see our new "Run Playbook" task listed, and on the right we'll see a "RUN" button. This will open a popup allowing us to enter a message or specify some options, but we can skip this and just press "RUN".

The result should look something like this:
<div align="center">
    <img src="readme-utils/semaphore-images/succesful-run.png" alt="Succesful task" width="738">
</div>

<div align="right">[ <a href="#table-of-contents">↑ Back to top ↑</a> ]</div>

## Running Ansible through the terminal[![](https://raw.githubusercontent.com/aregtech/areg-sdk/master/docs/img/pin.svg)](#ansible-terminal) 
Another, less-straightforward way to run Ansible is to use the Ansible CLI commands. In this section, we will cover how to get into the Ansible controller container and run playbooks through the terminal.

First, we need to get a terminal operating inside `ansible-container`. We can do this by running the following docker command:
```bash
$ docker exec -it ansible-container /bin/bash
root@c02102bd917f:/ansible#
```

Now that we're inside the container, we can execute Ansible commands. We're interested in running playbooks, so we'll use `ansible-playbook`. Here's the syntax:

```
ansible-playbook -i <inventory_file> <playbook_file>
```

If we inspect the files in this container, we'll find that this project's `inventory.yml` and `playbook.yml` can already be found in the root folder, so to run said playbook with said inventory we can simply run:

```bash
$ ansible-playbook -i inventory.yml playbook.yml
```

Once Ansible is done configuring our hosts, we'll get a summary on what's changed. Since playbooks are idempotent, you don't have to worry about running a playbook multiple times.
