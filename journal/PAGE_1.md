# PAGE 1

## Date: 30.04.2026, 11:51 A.M. EET

Today marks an important milestone achieved. Today I managed to link the storage configuration to the PostgreSQL and 
pgBackRest configuration. Everything executed perfectly. At the moment, after the execution of the 
`install_and_configure_postgresql.yml`, the system:

- has storage configured: 4 LVMs (created from the same VG) for the data, backups, logs and wal
- registers the system, attaches a subscription, updates dnf, installs the RPM PostgreSQL repo and 
enables the EPEL repository (needed for installation of the `libssh2` package - needed for the pgBackRest)
- installs PostgreSQL and configures it so that it can use the LVMs created (data, logs, wal)
- configures pgBackRest and makes it save the backups inside the backups LVMs created at the first step


## Things to improve (2nd execution crashed)

Even though the playbook executed flawlessly at the first iteration, at a second execution it fails. The problem comes 
from the fact that the tasks that manage the LVMs creation are not idempotent. I need to skip the steps of calculating 
the Free space and recreate the LVMs with the new size (this leads to expand or shrink of the LVMs which fails). 

I need to create new tasks and conditions that impose the execution of these tasks when new disks are added. If there 
are no empty disks detected, we should check only if the LVMs are created and that is it. 


## What to do for the rest of the day

Document the process and work on the styling of the playbooks.

## Future tests

Test if the storage management part works when adding new disks.
