# ROADMAP - PostgreSQL Backup Automation

## 🌱 Phase 0: Storage Management
- [ ] Auto-detect when new disks are added
- [ ] Auto-configure the disks to create/extend LVMs for PostgreSQL: data, logs, backups, wal
  - [x] Create n equal-sized primary partitions when new disk detected
  - [ ] Create PVs based on the new partitions
  - [ ] Create VGs (1 main and 1 snapshots)
  - [ ] Create LVMs
  - [ ] Auto mount persist (fstab)
- [ ] Remove LVMs, VGs, PVs, primary partitions
  - [ ] Remove primary partitions
    - [x] All partitions
    - [ ] Specific partitions
  - [ ] Remove PVs
    - [ ] All PVs
    - [ ] Specific PVs 
  - [ ] Remove VGs
    - [ ] All VGs
    - [ ] Specific VGs
  - [ ] Remove LVMs 
    - [ ] All LVMs
    - [ ] Specific LVMs
- [ ] Create snapshots
- [ ] Extend PVs, VGs and LVMs
  - [ ] Extend PVs
  - [ ] Extend VGs
  - [ ] Extend LVMs
- [ ] Shrink PVs, VGs, LVMs
  - [ ] Shrink PVs
  - [ ] Shrink VGs
  - [ ] Shrink LVMs


## ✅ Phase 1: Solid base
- [x] PostgreSQL Install
- [x] Start/Stop PostgreSQL service using the best practices 
- [x] pgBackRest install + config
- [x] Template for `postgresql.conf`
- [x] Config `pgbackrest.conf` (backup retention, repo, file bundling, block incremental, change settings)
- [x] Ansible tasks for backup full/incremental/differential
- [x] Jobs in cron via Ansible
- [x] Manual restore tested (full) -> see the `RESTORE.md`
- [x] Delta restore manually tested -> see the `RESTORE.md`
- [x] Restore a specific database (manual intervention) -> see the `RESTORE.md`
- [ ] PITR (Point-In-Time Recovery) manually tested -> see the `RESTORE.md`

## 🧩 Phase 2: Expansion and Refinement
- [ ] Restore via Ansible
- [ ] Delta restore, PITR, archive-restore
- [ ] Automatic backup validation
- [ ] Testing with corrupted database or missing WAL files

## 🔧 Phase 3: Automation and Tools
- [ ] Pyhton script for backup monitoring
- [ ] Email alert if a backup fails
- [ ] Export backup status in JSON / CSV format
- [ ] Custom commands: `pg_executor backup diff`

## 🌐 Phase 4: Web Interface + API
- [ ] Flask application for visual status overview
- [ ] REST API for backup status
- [ ] Trigger backups from the interface

## 🔄 Phase 5: Replication and High Availability
- [ ] Read-only replica
- [ ] Failover/switchover testing
- [ ] Replica lag monitoring
