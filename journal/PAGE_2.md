# PAGE 2

## Date: 02.05.2026, 18:51 EET


Today I managed to make the `storage_management > storage_device_auto_detect` task idempotent. I moved the LVM logic inside the 
`configure_logical_volumes.yml`. There, I created 3 cases:
- the LVMs are already created
- not all the LVMs are created
- no LVMs were created at all (the initial logic)

This way I managed to make the LVM configuration idempotent

## Interesting things 

While debugging my code, I found the following thing:

```ansible
# If we do not initialize the 'existing_lvms' list,
# if the 'lvs_output.stdout_lines[2:]' list is empty,
# then the 'Create list with existing non-standard LVMs'
# will not execute, and we will have existing_lvms undefined

- name: "Initialize 'existing_lvms' list"
  ansible.builtin.set_fact:
    existing_lvms: []

- name: "Create list with existing non-standard LVMs"
  ansible.builtin.set_fact:
    existing_lvms: "{{ existing_lvms | default([]) + [
      item.split()[0]
    ] }}"
  loop: "{{ lvs_output.stdout_lines[2:] }}"
  when: lvs_output is defined
```

I need to mention the task `Initialize 'existing_lvms' list` task is the solution of this problem.

What this is actually about? This is about the fact that the `'Create list with existing non-standard LVMs'` and all 
subsequent tasks were skipped. Why? Because the `lvs_output.stdout_lines[2:]` was empty and the task was not execution 
because the loop did not have elements. If the `lvs_output.stdout_lines[2:]` is empty that means that the 
`existing_lvms` is not created. That is why I initialized the `existing_lvms` before. 
 

# TO DO

Need to parameter the provisioning of the Storage and PostgreSQL database. Ideas to parameter:
- storage disk
- postgresql.conf options
- pg_hba.conf options

This parameters should be contained in 3 profiles: 
- small
- medium
- large

Before I start implementing these profiles I need to implement this flow: 

- master node connects to the host where the hypervisor (Oracle VirtualBox) and : 
  - activates OpenSSH Server
  - start the sshd service
  - enables the sshd service
  - configures firewall
- execute VBoxManage commands in order to create a new VM using a specific base 
(a base that contains at least one empty disk)
- updates the Ansible inventory with the IP Address of the new VM
- starts configuring the new VM with the storage - PostgreSQL - pgBackRest flow we already have


