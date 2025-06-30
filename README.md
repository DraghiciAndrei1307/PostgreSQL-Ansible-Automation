# :wrench:PostgreSQL-Ansible-Control:wrench:

## Short Description

Playbook used for **installation**, **configuration** and **management** of the **PostgreSQL 13** service on the slave nodes. 

## Prerequisites

:white_check_mark: **Ansible version: core 2.14.18**

:white_check_mark: **Operating System: RHEL 9.5**

:white_check_mark: **Access SSH** 

## Usage instructions 

**First of all**, you need an inventory and a vault file for this to work. The inventory file should contain only the aliases for the slave nodes and the ansible_hostname for each of them (e.g. **192.168.xxx.xxx**). You can check that by using **ip a** command on the slave node terminal. The vault file should contain the **ansible_password**, the **ansible_become_password** and the **ansible_user**. The **ansible_user** and the **ansible_password** are needed for the **SSH** connection when ansible plays are being executed. The **ansible_become_password** is used to execute tasks that require sudo privileges. One more thing to add here is that the vault file also contains the RedHat credentials used for checking the **subscritpion-manager** status and, in case it is **unregistered**, to **register** it.   

**The structure of the vault file is:**

ansible_user: xxxxxx
ansible_password: xxxxxx
ansible_become_password: xxxxxx

redhat_username: "xxxxxx"
redhat_password: "xxxxxx"

After we ensured the vault and the inventory file, we can move on to how to use the playbooks. 

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

The `ansible.cfg` file is a configuration file used by Ansible for setting the **"rules"** for the playbooks. There are **3 places** where you can find or create the `ansible.cfg` file:

1. the folder where you run the playbooks: `ansible.cfg`
2. the home directory of the user: `~/.ansible.cfg`
3. in the global location of the system: `/etc/ansible/ansible.cfg`

The one that has priority is the one inside the folder you are running the playbook. 

Inside 





