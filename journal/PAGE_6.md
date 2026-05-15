# PAGE 6

## time: 15.05.2026, 12:10 EET 

Today I am thinking of structuring my architecture in order to be more robust and more efficient. 

What I plan to do today is to create the following structure. If the hypervisor (the place where the VMs are hosted) 
does not contain any base VM, it should create a RedHat-Base VM which does not contain anything. 

After, using that base VM, there should be created 3 more base vms (bronze, silver and gold which contain 
1, 2, 3 disks of 10GB each) where I shall configure storage, install PostgreSQL and pgBackRest. 

When a user wants to create a specific VM, it will just clone one of the base VMs (bronze, silver or gold) and the 
process will be rapid. 

Before I start creating such architecture...I need to do one thing. I need to create some CI/CD where the compile and 
build process of the pgBackRest is performed and results in the creation of a release. 
