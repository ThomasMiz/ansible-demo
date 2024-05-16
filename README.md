
# Docker commands

In the root of the project, run docker compose.

```bash
$ docker-compose up -d
```

Connect to the ansible-container
```bash
$ docker exec -it ansible-container /bin/bash
```

Test connection. Add as many host you want in the inventory.ini file
```bash
root@f1fe1a8fdfd7:/ansible$ ansible -m ping all
ubuntu-container1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
ubuntu-container2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

There is a testing-playbook that installs postgresql in a remote node if you want to try it
```bash
root@f1fe1a8fdfd7:/ansible$ ansible-playbook testing-playbook.yml

PLAY [Install PostgreSQL] *********************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [ubuntu-container2]
ok: [ubuntu-container1]

TASK [Install PostgreSQL] ***********************************************************************************************************************************************
changed: [ubuntu-container1]
changed: [ubuntu-container2]

TASK [Verify PostgreSQL installation] **********************************************************************************************************************************
changed: [ubuntu-container2]
changed: [ubuntu-container1]

TASK [Display a successful installation message if PostgreSQL is installed] ********************************************************************************************
ok: [ubuntu-container1] => {
    "msg": "PostgreSQL installed successfully. Version: psql (PostgreSQL) 16.2 (Ubuntu 16.2-1ubuntu4)"
}
ok: [ubuntu-container2] => {
    "msg": "PostgreSQL installed successfully. Version: psql (PostgreSQL) 16.2 (Ubuntu 16.2-1ubuntu4)"
}

PLAY RECAP *************************************************************************************************************************************************************
ubuntu-container1          : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu-container2          : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

# Running reddy

Now let's run the project:

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