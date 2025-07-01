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
- `--ask-vault-password` option is used in order to provide the password for the vault file the playbook is linked to (check the `vars_files` inside any playbook present inside this repo to understand how to link the vault to the playbook)

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

When we run the playbook, the SSH connection happens. When the SSH connection is established, basically there will be a check of the SSH key of the host we want to connect to. By putting the above inside the ansible.cfg file, we specify that we want to skip this step to make the process easier. Why that? If the SSH key is changed, the check will return an error and the playbook execution will stop, requiring manual intervention to accept or update the new key. This can be disruptive, especially in dynamic environments like cloud-based infrastructure, where hosts may be frequently recreated or have their SSH keys regenerated. By disabling host key checking, we allow Ansible to proceed without blocking, ensuring smoother and uninterrupted automation. However, note that this reduces SSH security and should be avoided in production environments where host identity verification is important.

### The `postgresql_install.yml` playbook description

As the name file says, this playbook is used for installing PostgreSQL on the slave nodes. 

The code of the `postgresql_install.yml` playbook can be observed below:

```bash
---
- name: "Install, Initialize and Start PostgreSQL service"
  hosts: dbs
  become: yes
  vars_files:
    - group_vars/vault.yml
  vars:
    pg_version: 13
  tasks:
    - name: "DNF: Update all packages (optional)"
      ansible.builtin.dnf:
        name: "*"
        state: latest
        update_only: yes
        nobest: yes

    - name: "Install the repository RPM"
      ansible.builtin.dnf:
        name: "https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm"
        state: present
        disable_gpg_check: yes

    - name: "Install PostgreSQL version {{ pg_version }}"
      ansible.builtin.dnf:
        name: postgresql{{ pg_version }}-server
        state: present

   # - name: "Copy PgBackRest build on the slave nodes "

   # - name: "Install pgBackRest"
   #   ansible.builtin.yum:
   #     name: pgbackrest
   #     state: present

    - name: "Initialize the database"
      ansible.builtin.command: "/usr/pgsql-{{ pg_version }}/bin/postgresql-{{ pg_version }}-setup initdb"
      args:
        creates: /var/lib/pgsql/{{ pg_version }}/data/PG_VERSION

    - name: "Enable automatic start"
      ansible.builtin.service:
        name: postgresql-{{ pg_version }}
        enabled: yes

    - name: "Start the PostgreSQL service"
      ansible.builtin.service:
        name: postgresql-{{ pg_version }}
        state: started

```

The playbook above contains one "play" called "Install, Initialize and Start PostgreSQL service". This play is executed only once when you run the playbook using the following command: `ansible-playbook -i inventory postgresql_install.yml --ask-vault-password`. This play has the following options:
- it is applied to all the hosts in the group called `dbs`. This was set here: `hosts: dbs`. This means that the play will target all the hosts in the group `dbs`. If you do not want to target a **group**, but **only one host**, you can put the alias of that host in there (e.g. `hosts: postgresql-node1`; see the inventory example in the ).
- can use **sudo** when needed. The option `become: yes` means that the tasks contained by the play can be executed as **sudo** when needed. For this to work, you will need the **sudo password** and the **user** you are targeting in the vault file at `ansible_user`.

To access the variables that we want to keep **hidden** in the **vault file**, we have to include the path to the **vault** file in the `vars_files`. The `vars_files` contains in general all the paths to the files in which we stored important variables needed by the play inside the playbook when executed. In our case, I've included only the path to the vault file which is: `- group_vars/vault.yml`.

Another way to create and use variables is to create them at the play level. This was perfectly exemplified here: 

```bash
- vars:
    - pg_version: 13
```

Here we created one variable the **pg_version** variable with the 13 value stored inside it. We are using this variable to specify the **PostgreSQL major version** that we want to install on our slave nodes.

Now, we will talk about tasks. Inside this play, we have several tasks, each with his name: 
- DNF: Update all packages (optional)
- Install the repository RPM
- Install PostgreSQL version {{ pg_version }}
- Install pgBackRest
- Initialize the database
- Enable automatic start
- Start the PostgreSQL service

Each task from the play above has a terminal command with the same functionality. Basically, if you want to create a play (a sequence of tasks) in Ansible, you just need to know the terminal commands and the order in which they must be executed for the desired outcome.

For the play above, the corresponding terminal commands are: 

```bash
# Update the  dnf repos
dnf update --nobest -y

# Install the official PostgreSQL repo
dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# Install the PostgreSQL from the official repo we installed above
dnf install postgresql13-server

# Init the PostgreSQL database
postgresql-setup --initdb

# Enable and Start the PostgreSQL service
systemctl enable postgresql13-service
systemctl start postgresql13-service
```
Mentions: 
- In the code snippet above I used `postgresql13-server`. You can use any version of PostgreSQL you want. 
- Before you write an Ansible script, focus on what the terminal commands are. After you can move on to translating it into YML format.  

We will start addressing each task starting now.

The first task is updating the packages to be up-to-date. Why do we care to have the packages up-to-date? If the packages were not updated for some time:
- you cannot install the desired package
- you will install an old version of the package that is not compatible with the system
- your system will have security vulnerabilities, bugs and exploits - which are solved by the patch  

The syntax of the first task is: 

```bash
- name: "DNF: Update all packages (optional)"
  ansible.builtin.dnf:
    name: "*"
    state: latest
    update_only: yes
    nobest: yes
```

For this task, we are using the builtin module of the ansible, module which is called: dnf
This module has the following arguments: name, state, update_only, nobest. `name: "*"` means that we want to update all the **installed** packages, `state: latest` means that the targeted packages will be updated to the latest version, `update_only: yes` tells Ansible to not install new packages, only update the installed ones, the ones present in the repo. `nobest: yes` means that the dnf will update the focused packages to the last approved version, not the best one, but the one that works. 

The second task is to install the official PostgreSQL repo from which we will install the PostgreSQL. The syntax of the second task is:

```bash
- name: "Install the repository RPM"
      ansible.builtin.dnf:
        name: "https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm"
        state: present
        disable_gpg_check: yes
```

We are using the dnf module to install the rpm file from the PostgreSQL official repo link. `state: present` checks if the `.rpm` file is present on our file system. If it is not, the package will be installed (this is called **idempotency**). When installing from an external source, the **dnf** checks if that the GPG key is present on our system. If you do not have this key, usually the dnf will skip the installation. In order to disable this check, we use: `disable_gpg_check: yes`.   

The third task installs the PostgreSQL from the official repo only if it wasn't already installed (`state: present`):

```bash
- name: "Install PostgreSQL version {{ pg_version }}"
  ansible.builtin.dnf:
    name: postgresql{{ pg_version }}-server
    state: present
```

The fourth task initializes the database. The syntax of the task is displayed below:

```bash
- name: "Initialize the database"
  ansible.builtin.command: "/usr/pgsql-{{ pg_version }}/bin/postgresql-{{ pg_version }}-setup initdb"
  args:
    creates: /var/lib/pgsql/{{ pg_version }}/data/PG_VERSION
```

We are using here the `command` module of the Ansible to run the following command: 

```bash
/usr/pgsql-13/bin/postgresql-13-setup initdb
```

After we run this command, the `/var/lib/pgsql/13/data` folder is created. 

The following code snippet ensures that if the file PG_VERSION already exists on our file system, the database was already initialized, which means this task can be skipped (idempotency).  

```bash
  args:
    creates: /var/lib/pgsql/{{ pg_version }}/data/PG_VERSION
```

The last two tasks ensure that the service is enabled and started. For these tasks, we are using the service Ansible module to work with processes. `enabled: true` checks if the service is enabled or not. If it was already enabled, this task is skipped. If not, the service is enabled. `state: started` checks if the process is running (active) or not (inactive). If the service was already started, the task is skipped. If not, the service is started.

### The `start_postgresql_service.yml` playbook description

This playbook is used for checking the PostgreSQL service status. If the status is **inactive**, the process will be started using systemctl.

The code of the playbook can be view in the code snippet below: 

```bash
---
- name: "Start PostgreSQL service"
  hosts: dbs
  vars_files:
    - group_vars/vault.yml
  vars:
    pg_version: 13
  become: yes
  tasks:
    - name: "Check Postgresql-{{ pg_version }}"
      shell: "systemctl status postgresql-{{ pg_version }}"
      register: check_output
      failed_when: false

    - name: "Start service"
      ansible.builtin.service:
        name: postgresql-{{ pg_version }}
        state: started
      when: '"Active: inactive (dead)" in check_output.stdout'
```

### The `stop_postgresql_service.yml` playbook description

This playbook is used for checking the PostgreSQL service status. If the status is **active**, the process will be stopped using `systemctl`.

The code of the playbook can be view in the code snippet below: 

```bash
---
- name: "Stop PostgreSQL service"
  hosts: dbs
  vars_files:
    - group_vars/vault.yml
  vars:
    pg_version: 13
  become: yes
  tasks:
    - name: "Check Postgresql-{{ pg_version }}"
      shell: "systemctl status postgresql-{{ pg_version }}"
      register: check_output

    - name: "Stop service"
      ansible.builtin.service:
        name: postgresql-{{ pg_version }}
        state: stopped
      when: '"Active: active" in check_output.stdout'
```

### The `register_system.yml` playbook description

This playbook is used to check if the VM has the RedHat subscription enabled. If not, the system will be registered.

The code snippet of the `register_system.yml` can be view below: 

```bash
---
- name: "Register system"
  hosts: localhost
  become: yes
  vars_files:
    - group_vars/vault.yml
  tasks:
    - ansible.builtin.shell: >
        subscription-manager register
        --username={{ redhat_username }}
        --password={{ redhat_password }}
        --auto-attach
      register: register_output
      failed_when: "'Invalid username or password' in register_output.stderr"

    - ansible.builtin.shell: "subscription-manager attach --auto"

```

### The `pgbackrest_build_process.yml` playbook description

As the RHEL pgBackRest documentation says: "When building from source it is best to use a build host rather than building on production. Many of the tools required for the build should generally not be installed in production. pgBackRest consists of a single executable so it is easy to copy to a new host once it is built." (Link: `[pgBackRest Documentation](https://pgbackrest.org/user-guide-rhel.html#build)`). So, basically, we are using in this case the ansible-master node in order to create the executable that we will copy to the slave-nodes and install the pgBackRest there.

The Ansible code snippet can be observed below: 

```bash
---
- name: "pgBackRest build process"
  hosts: localhost
  become: yes
  vars:
    dependencies:
      - meson
      - gcc
      - postgresql13-devel
      - openssl-devel
      - libxml2-devel
      - lz4-devel
      - libzstd-devel
      - bzip2-devel
      - libyaml-devel
      - libssh2-devel

  vars_files:
    - group_vars/vault.yml
  tasks:
    - name: "Create build directory"
      command: mkdir -p /pgBackRest_build

    - name: "Download into /pgBackRest_build"
      ansible.builtin.shell: "wget -q -O - https://github.com/pgbackrest/pgbackrest/archive/release/2.55.1.tar.gz | tar zx -C /pgBackRest_build"

    - name: "Update DNF"
      ansible.builtin.yum:
        name: "*"
        state: latest
        update_only: yes
        update_cache: true
        nobest: yes

    - name: "Enable codeready-builder(CRB) repo"
      ansible.builtin.command: >
        subscription-manager repos --enable codeready-builder-for-rhel-9-x86_64-rpms
      when: ansible_distribution == "RedHat" and ansible_distribution_major_version == "9"

    - name: "Install EPEL repository (RHEL 9)"
      block:
        - name: "Download and install EPEL release RPM"
          ansible.builtin.dnf:
            name: "https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm"
            state: present
            disable_gpg_check: yes

        - name: "Import EPEL GPG key"
          ansible.builtin.rpm_key:
            key: "https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-9"
            state: present

    - name: "Install build dependencies"
      #debug:
      #  msg: "Installing package {{ item }}"
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: present
      loop: "{{ dependencies }}"

    - name: "Configure pgBackRest"
      ansible.builtin.shell: >
        PKG_CONFIG_PATH=/usr/pgsql-13/lib/pkgconfig
        meson setup /pgBackRest_build/pgbackrest /pgBackRest_build/pgbackrest-release-2.55.1
      args:
        creates: "/pgBackRest_build/pgbackrest/build.ninja"


    - name: "Build pgBackRest"
      ansible.builtin.shell: "ninja -C /pgBackRest_build/pgbackrest"

```
