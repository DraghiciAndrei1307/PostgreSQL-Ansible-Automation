# PAGE 4

## Date: 09.05.2026, 14:09 EET

## Results:

Today I managed to perform the following: 
- after obtaining the IP address of the newly created VM, I add it into a special 
inventory `~/PostgreSQL-Ansible-Automation/ansible/inventories/slave_nodes_inventory`
- after adding the new host to the `slave_nodes_inventory`, I copied the SSH public key from the master to the 
newly created slave
- After creating the slave, I managed to put everything (VM creation, storage config, PostgreSQL config and pgBackRest 
config) inside a special playbook named `provision_postgresql_VM.yml`. 


## Observations/TO-DOs:

- I am thinking to create a procedure (maybe in the future) that configures the whole project (config of the master 
node and the connection between the master node and the hypervisor host)
- At the moment, the master node, the hypervisor node and the connection between them was configured 
manually (procedure is documented here: [CON_WINDOWS_VIA_SSH_KEY.md](../docs/CON_WINDOWS_VIA_SSH_KEY.md))
- need to work on the idempotency of the whole procedure (VM - storage - database - pgBackRest) because 100% I am 
sure that if I run this a second time it will crash
- also, I need to make this procedure faster. What I am thinking is that I can:
  - compress and archive the PGDATA folder and just uncompress and unarchive it when new installation is performed
  - also, I can move the build process of the pgBackRest into CI/CD or something like that and I can just load that 
  into the newly created slave
- I need to update the Python CLI solution to use the Ansible procedures of this repo. The development of the new CLI 
should be on fast-forward as I do not have time at the moment. I need to create a Django API that uses the Python CLI 
for the provisioning of the PostgreSQL VMs.   






