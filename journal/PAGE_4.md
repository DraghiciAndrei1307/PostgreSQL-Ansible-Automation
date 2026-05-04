# PAGE 4

## Date: 04.05.2026, 21:09 EET

## Results:

Today I managed to clone a vm (command used: `vboxmanage clonevm...`) that was already existing on the windows host 
and obtain the IP address of the newly created VM. The last part was the hardest...after I managed to create the VM, 
I tried to use VBoxManage to obtain the IP Address, but with no success. The workaround I managed to use and automate
was to enable the `EPEL repo` on the `master node` and then install a tool named `arp-scan` (this procedure was 
performed manually). This tool helped me scan the local network and obtain the IP address of the newly created VM. 

The roles were not fully developed. There will be a refactor the next days in order to make the code and everything 
more organized.  

## What I still need to do on the VM provisioning

- use the obtained IP address in order to update the inventory and configure connection using SSH keys
- create the 3 base VMs needed to create profiles
- link the VM provisioning part to the storage - PostgreSQL - pgBackRest config part

