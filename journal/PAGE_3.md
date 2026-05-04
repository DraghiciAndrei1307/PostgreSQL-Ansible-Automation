# PAGE 3

# Plan

Today I am thinking:
- to create the connection between the master node and the windows host 
(where the VirtualBox hypervisor runs)
- create 3 base VMs which:
  - have attached designated size disks, in order to create the 3 profiles
  - have executed upon the `common` role:
    - have dnf updated
    - have the RPM repo already installed
    - have the EPEL enabled


# What I managed to do

- create the connection between the master node and the windows host 
(where the VirtualBox hypervisor runs): [CON_WINDOWS_VIA_SSH_KEY.md](../docs/CON_WINDOWS_VIA_SSH_KEY.md)

- execute Ansible role which lists the VMs that exist on the Windows host
