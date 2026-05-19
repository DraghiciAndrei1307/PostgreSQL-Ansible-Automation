# PAGE 8

## time: 19.05.2026, 16:47 EET 

Today I achieved an important milestone. I managed to create the "golden images" base VMs (bronze, silver and gold) and 
managed to clone them in order to obtain a VM configured with PostgreSQL and pgBackRest. 

What I managed to do in the last period and maybe I did not have the change to document was:
- Modified the provisioning procedure and made it faster by creating a CI/CD workflow which deals with the creation of 
a release that contains the build of the pgBackRest; This is important because I am not performing the build on the 
master node anymore and also this feature saves time (aprox. 10 minutes or so)
- Took the long procedure of provisioning (VM creation + Storage config + PostgreSQL config + pgBackRest config), which 
took general like 45 minutes or so and used it in creating 3 "golden images" which I will later use in creation of new 
VMs provisioned with PostgreSQL and pgBackRest. The thing with these "golden images" is that they help me reduce the 
provisioning time from ~45 minutes to ~5-10 minutes. I perform the procedure once and then I am just cloning the base 
VMs.   
- Used Vagrant which is a wrapper around the Virtual Box in order to simplify the process of configuring the base VMs 
(before the provisioning process starts). By configuring I mean things like Network Adapter to be on Bridged, create 
"student" user, create and attach disks, etc.
- After I obtained the "golden images", I created a new playbook which deals with the cloning, updating the inventory 
and copying the SSH key from master to the newly created (cloned) VM. 

In the end, I used the Django API in order to create a POST request and start the whole procedure. It was a success. A 
clone named "bronze_1" was created.

## Limitations for the moment

There is still the need to link the creation of the "golden images" base VMs to the cloning. At the moment, if you 
perform a POST request want to create a new VM (basically try to clone one of the bases) and you do not have the 
"golden" images configured on the hypervisor host, then the process will fail.

But, for the moment, the actual configuration where you configure the "golden images" separately in order to have them 
on the hypervisor host system, before you start making POST requests to clone them, will do just fine for the demo 
purposes. 

In the future, is common sense that I will fill every noticeable logic gap, but at the moment the time is not working in
my favor. 

## TO-DO for next days

- I need to work on the details. At the moment, the status of the VM creation (retrieved by the GET request performed over 
the Django endpoint) is not correlated to the actual process. Also, I need to work on the final status. The Celery 
worker says 'success' and the status saved into the SQLite3 database is Failed. So, the idea is that I need to be 
precise when it comes to the "provisioning" status.
- Another thing I need to do is to send the name, base reference and other related stuff so that I can properly choose 
from the "golden images" and customize the new VM with a name (given by the user) and so on...
- Create an in-depth report of this project which contains:
  - current status
  - stack used
  - what features does it have
  - limitations
  - how to use (what you need to configure)
  - future improvements
