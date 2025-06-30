# :wrench:PostgreSQL-Ansible-Control:wrench:

## Short Description

Playbook used for **installation**, **configuration** and **management** of the **PostgreSQL 13** service on the slave nodes. 

## Contents:

- Prerequisites
- Usage Instructions
- Project/Repo contents
    - The `ansible.cfg` configuration file description
    - The `postgresql_install.yml` playbook description

## Prerequisites

:white_check_mark: **Ansible version: core 2.14.18**

:white_check_mark: **Operating System: RHEL 9.5**

:white_check_mark: **Access SSH** 

:white_check_mark: **inventory and vault files** 

After you created yourself a **RedHat 9.5 VM** with **Ansible** and configured the **SSH access**, you need an **inventory** and a **vault** file for this to work. The inventory file should contain only the **aliases** for the slave nodes and the **ansible_hostname** for each of them (e.g. **192.168.xxx.xxx**). You can check that by using **ip a** command on the slave node terminal. The vault file should contain the **ansible_password**, the **ansible_become_password** and the **ansible_user**. The **ansible_user** and the **ansible_password** are needed for the **SSH** connection when ansible plays are being executed. The **ansible_become_password** is used to execute tasks that require **sudo privileges**. One more thing to add here is that the **vault.yml** file also contains (in this case) the RedHat credentials used for checking the **subscritpion-manager** status and, in case it is **unregistered**, to **register** it.   

**The structure of the `groups_vars/vault.yml` file is:**

```bash
ansible_user: xxxxxx
ansible_password: xxxxxx
ansible_become_password: xxxxxx

redhat_username: "xxxxxx"
redhat_password: "xxxxxx"
```

An example of the `inventory` file can be observed below: 

```bash
postgresql-node1 ansible_host=xxx.xxx.xxx.xxx
postgresql-node2 ansible_host=xxx.xxx.xxx.xxx
postgresql-node3 ansible_host=xxx.xxx.xxx.xxx

[dbs]
postgresql-node1
postgresql-node2
postgresql-node3
```

If you want to modify the vault file, you can use the following: 

```bash
ansible-vault edit /path/to/vault/file.yml
```

After we ensured the vault and the inventory file, we can move on to talk about how to use the playbooks inside this project.

## Usage Instructions

This is how you should run the playbooks (the .yml files): 

```bash
ansible-playbook -i inventory name_of_the_playbook.yml --ask-vault-password
```

As you can see, the following options were used:

- `-i` option is used for linking the inventory file to this playbook run
- `--ask-vault-password` option is used in order to provide the password for the vault file the playbook is linked to (check the `vars_files` inside any playbook present inside this repo in order to understand how to link the vault to the playbook)

Another option that you can use is the `--check` option. What this does is that it lets you run the playbook without making any changes on the slave node.

## Project/Repo contents

The project contains the following: 

- ansible.cfg
- inventory
- stop_postgresql_service.yml
- start_postgresql_service.yml
- postgresql_install.yml
- register_system.yml
- pgbackrest_build_process.yml
- group_vars/
    - vault.yml

### The `ansible.cfg` configuration file description

The `ansible.cfg` file is a configuration file used by Ansible for setting the **"rules"** for the playbooks. There are **3 places** where you can find or create the `ansible.cfg` file:

1. the folder where you run the playbooks: `ansible.cfg`
2. the home directory of the user: `~/.ansible.cfg`
3. in the global location of the system: `/etc/ansible/ansible.cfg`

The one that has priority is the one inside the folder you are running the playbook, after that comes the one inside the user's home directory and the last, but not the least, is the one from `/etc/ansible/ansible.cfg` 

Inside the `ansible.cfg` file (the one inside our repo), we have the following: 

```bash
[defaults]
host_key_checking=False
```

When we run the playbook, the SSH connection happens. When the SSH connection is established, basically there will be a check of the SSH key of the host we want to connect to. By putting the above inside the ansible.cfg file, we specify that we want to skip this step in order to make the process easier. Why that? Because if the SSH key is changed, the check will return an error and the playbook execution will stop, requiring manual intervention to accept or update the new key. This can be disruptive, especially in dynamic environments like cloud-based infrastructure, where hosts may be frequently recreated or have their SSH keys regenerated. By disabling host key checking, we allow Ansible to proceed without blocking, ensuring smoother and uninterrupted automation. However, note that this reduces SSH security and should be avoided in production environments where host identity verification is important.

### The `postgresql_install.yml` playbook description

Now, `postgresql_install.yml` playbook


