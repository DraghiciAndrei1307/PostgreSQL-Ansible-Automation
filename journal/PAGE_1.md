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


## Execution Flow

1. Detect empty disks
2. Create partitions (XFS)
3. Create PVs
4. Create VG (vg_postgres)
5. Create LVMs:
   - lvm_pg_data
   - lvm_pg_wal
   - lvm_pg_logs
   - lvm_pg_backups
6. Format with XFS
7. Mount to:
   - /opt/postgresql/data
   - /opt/postgresql/wal
   - /opt/postgresql/logs
   - /opt/postgresql/backups
8. Install PostgreSQL
9. Initialize database in custom PGDATA
10. Configure WAL (symlink)
11. Configure pgBackRest
12. Run backup

## Things to improve (2nd execution crashed)

Even though the playbook executed flawlessly at the first iteration, at a second execution it fails. The problem comes 
from the fact that the tasks that manage the LVMs creation are not idempotent. I need to separate initial provisioning 
from subsequent runs (skip the steps of calculating the Free space and recreate the LVMs with the new size this leads 
to expand or shrink of the LVMs which fails). 

I need to create new tasks and conditions that impose the execution of these tasks when new disks are added. If there 
are no empty disks detected, we should check only if the LVMs are created and that is it. 

## What to do for the rest of the day

Document the process and work on the styling of the playbooks.

## Future tests

Test if the storage management part works when adding new disks.

## Known Issues

- LVM creation is not idempotent
- Second run attempts to resize existing LVMs
- Shrinking is not allowed without force=true
