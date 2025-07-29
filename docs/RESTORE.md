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

There may be cases in which you want to restore just some databases, not all of them. 

In this case, the procedure is as follows:

- Create 2 databases
```

sudo -u postgres psql -c "create database test1; "
sudo -u postgres psql -c "create database test2: "

```

- Create and populate the tables inside each one of them

```

sudo -u postgres psql -c "create table test1_table (id int);
                          insert into test1_table (id) values (1); " test1
                          
sudo -u postgres psql -c "create table test2_table (id int);
                          insert into test2_table (id) values (2); " test2

```

- Perform an incremental backup

```

sudo -u postgres pgbackrest --stanza=demo --type=incr backup

```

- Check the size of the databases

It is a good practice to note which is the size of the databases. To do that, type the following command:

```
sudo -u postgres du -sh /var/lib/pgsql/13/data/base/*
```

You will see something like: 

```
8.0M    /var/lib/pgsql/13/data/base/1
8.0M    /var/lib/pgsql/13/data/base/14447
8.1M    /var/lib/pgsql/13/data/base/14448
8.0M    /var/lib/pgsql/13/data/base/16384
8.0M    /var/lib/pgsql/13/data/base/16385
```

These are the folders containing the data of our databases. Each folder is associated with a database. 
Folders are named using the OID of the databases. To see the OID of the database, there are 2 ways that I can show you:

- 1) After you performed the incremental backup, type the following: 
```
sudo -u postgres pgbackrest --stanza=demo --set=20250729-123102F_20250729-123206I info
```

This will give you the information about the desired incremental backup mentioned in the `--set=` option. In our case,
the `20250729-123102F_20250729-123206I` incremental backup performed earlier. 

The output should look like this: 

```
stanza: demo
    status: ok
    cipher: aes-256-cbc

    db (current)
        wal archive min/max (13): 0000000300000001000000A9/00000003000000010000

        incr backup: 20250729-123102F_20250729-123206I
            timestamp start/stop: 2025-07-29 12:42:17+03 / 2025-07-29 12:42:20+
            wal start/stop: 0000000300000001000000B3 / 0000000300000001000000B3
            lsn start/stop: 1/B3000028 / 1/B3000138
            database size: 40.1MB, database backup size: 4.2MB
            repo1: backup size: 41.7KB
            backup reference list: 20250729-124102F
            database list: postgres (14448), test1 (16384), test2 (16385)

```

As you can see, the last line contains the name of the databases and their corresponding OID.

- 2) Type the following:

```
sudo -u postgres psql -c "select oid, datname from pgdatabase;"

# It will output this:

  oid  |  datname
-------+-----------
 14448 | postgres
 16384 | test1
     1 | template1
 14447 | template0
 16385 | test2
(5 rows)
```

- Connect to the cluster and check the databases and the tables created 
```
postgres=# \l+
                                                                    List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   |  Size   | Tablespace |                Description
-----------+----------+----------+-------------+-------------+-----------------------+---------+------------+--------------------------------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                       | 8245 kB | pg_default | default administrative connection database
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +| 8081 kB | pg_default | unmodifiable empty database
           |          |          |             |             | postgres=CTc/postgres |         |            |
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +| 8081 kB | pg_default | default template for new databases
           |          |          |             |             | postgres=CTc/postgres |         |            |
 test1     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                       | 8237 kB | pg_default |
 test2     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                       | 8237 kB | pg_default |
(5 rows)

postgres=# \c test1
You are now connected to database "test1" as user "postgres".
LINE 1: select * from test1_database;
                      ^
test1=# \dt+
                                List of relations
 Schema |    Name     | Type  |  Owner   | Persistence |    Size    | Description
--------+-------------+-------+----------+-------------+------------+-------------
 public | test1_table | table | postgres | permanent   | 8192 bytes |
(1 row)

test1=# select * from test1_table;
 id
----
  1
(1 row)

test1=# \c test2
You are now connected to database "test2" as user "postgres".
test2=# select * from test2_table;
 id
----
  2
(1 row)

```

- Stop the cluster

```
# Check the status
sudo systemctl status postgresql-13

# Stop the cluster
sudo systemctl stop postgresql-13
```

- Restore only the `test1` database 

```
# Restore only the test1 db
sudo -u postgres pgbackrest --stanza=demo --delta --db-include=test1 --type=immediate --target-action=promote restore
```

The command above has some special options:
- 1) `--db-include=test1` -> this means to restore only the test1 database  
- 2) `--type=immediate` -> this option is used in order to avoid errors. When zeroed pages are 
restored (the pages of the test2 database), because they are empty they will return errors. To avoid this errors, 
we use this option.
- 3) `--target-action=promote`


- Start the cluster

```
# Start the cluster
sudo systemctl start postgresql-13
```

After that, we have to proceed with some checks:

Before: 
```
postgres=# \l+
                                                                    List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   |  Size   | Tablespace |                Description
-----------+----------+----------+-------------+-------------+-----------------------+---------+------------+--------------------------------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                       | 8245 kB | pg_default | default administrative connection database
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +| 8081 kB | pg_default | unmodifiable empty database
           |          |          |             |             | postgres=CTc/postgres |         |            |
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +| 8081 kB | pg_default | default template for new databases
           |          |          |             |             | postgres=CTc/postgres |         |            |
 test1     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                       | 8237 kB | pg_default |
 test2     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                       | 8237 kB | pg_default |
(5 rows)

```

After:
```
postgres=# \l+
                                                                    List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   |  Size   | Tablespace |                Description
-----------+----------+----------+-------------+-------------+-----------------------+---------+------------+--------------------------------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                       | 8245 kB | pg_default | default administrative connection database
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +| 8081 kB | pg_default | unmodifiable empty database
           |          |          |             |             | postgres=CTc/postgres |         |            |
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +| 8081 kB | pg_default | default template for new databases
           |          |          |             |             | postgres=CTc/postgres |         |            |
 test1     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                       | 8237 kB | pg_default |
 test2     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                       | 8089 kB | pg_default |
(5 rows)

```

Note that the test2 database has a smaller size than before. This is because the entire database was restored as
sparse, zeroed files. The WAL files were restored, but the essential data files were not restore, or they were restored,
but the important data is missing, they are zeroed files. 

Also, if we are trying to connect to the test2 database, we receive the following:
```
postgres=# \c test1
You are now connected to database "test1" as user "postgres".
test1=# select * from test1_table;
 id
----
  1
(1 row)

test1=# \c test2
FATAL:  relation mapping file "base/16385/pg_filenode.map" contains invalid data
Previous connection kept

```

As said before, the key files contain no data...

```
[postgres@postgresql-node1 ~]$ psql -c "select * from test2_table;" test2
psql: error: FATAL:  relation mapping file "base/16385/pg_filenode.map" contains invalid data
```

Other thing to observe:

Before: 

```
[postgres@postgresql-node1 base]$ du -sh /var/lib/pgsql/13/data/base/*
8.0M    /var/lib/pgsql/13/data/base/1
8.0M    /var/lib/pgsql/13/data/base/14447
8.1M    /var/lib/pgsql/13/data/base/14448
8.0M    /var/lib/pgsql/13/data/base/16384
8.0M    /var/lib/pgsql/13/data/base/16385
```

After: 

```
[postgres@postgresql-node1 ~]$ du -sh /var/lib/pgsql/13/data/base/*
8.0M    /var/lib/pgsql/13/data/base/1
8.0M    /var/lib/pgsql/13/data/base/14447
8.1M    /var/lib/pgsql/13/data/base/14448
8.1M    /var/lib/pgsql/13/data/base/16384
16K     /var/lib/pgsql/13/data/base/16385
```

See that the `test2 (16385)` has only `16K` size now compared to `8.0M` . 

- Drop the invalid database (This means test2)

Now that we check the results, the only thing to do is to drop the invalid database, using:
```
# Drop the invalid database (test2)
sudo -u postgres psql -c "drop database test2;"
DROP DATABASE
```

- Check the result

```
# Check the result
sudo -u postgres psql -c "select oid, datname from pg_database;"
  oid  |  datname
-------+-----------
 14448 | postgres
 16384 | test1
     1 | template1
 14447 | template0
(4 rows)

```

## ANSIBLE RESTORE PROCEDURE

This is basically a translation of the steps above from the CLI to the Ansible (YAML) syntax.
