# PostgreSQL-Ansible-Control

## Short Description

Playbook used for installation, configuration and management of the PostgreSQL 13 service on the slave nodes.

## Prerequisites

- [X] Ansible version: core 2.14.18
- [X] Operating System: RHEL 9.5
- [X] Access SSH 

## Usage instructions 

First of all, you need an inventory and a vault file for this to work.

The inventory file should contain only the aliases for the slave nodes and the ansible_hostname for each of them (e.g. 192.168.xxx.xxx). You can check that by using **ip a** command on the slave node terminal. 





