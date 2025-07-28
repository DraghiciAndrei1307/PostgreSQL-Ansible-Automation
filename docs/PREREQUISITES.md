# Prerequisites

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
