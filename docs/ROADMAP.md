# ROADMAP - PostgreSQL Backup Automation

## ✅ Phase 1: Solid base
- [x] PostgreSQL Install
- [x] Start/Stop PostgreSQL service using the best practices 
- [x] pgBackRest install + config
- [x] Template for `postgresql.conf`
- [x] Config `pgbackrest.conf` (backup retention, repo, file bundling, block incremental, change settings)
- [x] Ansible tasks for backup full/incremental/differential
- [x] Jobs in cron via Ansible
- [x] Manual restore tested (full)
- [x] Delta restore manually tested
- [] PITR (Point-In-Time Recovery) manually tested

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
