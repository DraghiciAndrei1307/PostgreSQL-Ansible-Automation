## MANUAL RESTORE PROCEDURE

Until now I tested the default/basic restore which is based on the last full backup. 

To check the backup information on a slave host, connect as postgres user (the database owner which was granted 
ownership and other privileges over pgbackrest). So, you should connect to that user.

To perform a default restore (or if you want to test the procedure), do this:

1) Stop the postgresql service

```
# Check the status of the postgresql-13 service
sudo systemctl status postgresql-13

# Stop the postgresql-13 service
sudo systemctl stop postgresql-13
```

**Observations:**

Before you stop the cluster, it is a good practice to perform a full backup. So, the procedure (the manual procedure) 
is as follows:

```
# Perform a full backup
sudo -u postgres pgbackrest --stanza=demo --type=full --log-level-console=info backup 

# Check the status of the postgresql-13 service
sudo systemctl status postgresql-13

# Stop the postgresql-13 service
sudo systemctl stop postgresql-13

```

2) Delete something from the `/var/lib/pgsql/13/data` folder in order to crash the cluster

3) Try to start the cluster

```

# Check the status of postgresql-13 service
sudo systemctl status postgresql-13

# Start the postgresql-13 service
sudo systemctl start postgresql-13

```

You will get an error which says that the cluster is crushed. The error may look like this:
```
Job for postgresql-13.service failed because the control process exited with error code.
See "systemctl status postgresql-13.service" and "journalctl -xeu postgresql-13.service" for details.
```

4) In order to solve this problem, we have a few options:

- **default option** (use the latest full backup)

```
# Clean the /var/lib/pgsql/13/data folder
sudo -u postgres find /var/lib/pgsql/13/data -mindepth 1 -delete

# Perform the restore
sudo -u postgres pgbackrest --stanza=demo restore

# Check the status the cluster/postgresql-13 service
sudo systemctl status postgresql-13

# Start the postgresql-13 service
sudo systemctl start postgresql-13
 
```
Everything should work now.

- **delta option** 

When using the delta option, we don’t need to clean the /var/lib/pgsql/13/data directory before restoring. 
The tool compares the files in the backup with those currently in the data directory using SHA-1 checksums. 
Only the files that differ are restored, which makes the process more efficient.  

You should proceed like this:

```

# After you tried to start the cluster and received that error, use the following:

sudo -u postgres pgbackrest --stanza=demo --delta --log-level-console=detail restore

# Start the cluster

sudo systemctl start postgresql-13

```

- PITR (Point-In-Time Recovery)

## ANSIBLE RESTORE PROCEDURE

This is basically a translation of the steps above from the CLI to the Ansible (YAML) syntax.
