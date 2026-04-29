# PROBLEMS_SOLUTION

When developing the storage provisioning procedure I encountered a problem related to GRUB that gave me the idea of 
documenting such cases. I think that these situations are worth saving for future reference.   

## Warning: Unmaintained driver is detected: e1000

### Context

When trying to make the storage_device_auto_detect role idempotent I had one error that occurred when creating the 
physical volumes. At the moment I left the problem unsolved and closes the VM. Later, when I tried to start the VM, I 
stumbled upon the following problem:

```terminaloutput
[     2.468130] Warning: Unmaintained driver is detected: e1000
[     2.478951] Warning: Unmaintained driver is detected: e1000_init_module
[     3.704864] vmwgfx 0000:00:02.0: [drm] *ERROR* vmwgfx seems to be running on an unsupported hypervisor
[     3.704870] vmwgfx 0000:00:02.0: [drm] *ERROR* This configuration is likely broken.
[     3.704874] vmwgfx 0000:00:02.0: [drm] *ERROR* Please switch to a supported graphics device to avoid problems.
You are in emergency mode. After logging in, type "journalctl -xb" to view system logs, "systemctl reboot" to reboot, 
"systemctl default" or "exit" to boot into default mode.

Cannot open access to console, the root account is locked.
See sulogin(8) man page for more details.

Press Enter to continue. 

```

To be honest, at the moment I have no idea to solve this, but I will research everything and try to document the 
solution.

### Solution

When starting the VM, when you find yourself into the GRUB menu, press `e`. You should see something like:

```terminaloutput
load_video
set gfxpayload=keep
insmod gzio
linux ($root)/vmlinuz-5.14.0-611.45.1.el9_7.x86_64 root=/dev/mapper/rhel-root \n
ro crashkernel=1G-2G:192M,2G-64G:256M,64G-:512M resume=/dev/mapper/rhel-swap \n
rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb quiet
initrd ($root)/initramfs-5.14.0-611.45.1.el9_7.x86_64.img $tuned_initrd
```

Replace `rhgb quiet` with `systemd.unit=emergency.target`

Press `Ctrl-X` or `F10` to boot.

If the `systemd.unit=emergency.target` approach does not work (for me it didn't), try 
to add in the same place `rd.break`

You will now be in emergency mode:


```terminaloutput
Generating "/run/initramfs/rdsosreport.txt"


Entering emergency mode. Exit the shell to continue.
Type "journalctl" to view system logs.
You might want to save "/run/initramfs/rdsosreport.txt" to a USB stick or /boot after mounting them and attach it to 
a bug report.


#switch_root:/#
```

Typed 

```terminaloutput
switch_root:/# lvm pvscan
    PV /dev/sda2 VG rhel  lvm2 [<19.00 GiB / 0   free]
    Total: 1 [<19.00 GiB] / in use: 1 [<19.00 GiB] / in no VG: 0 [0   ]
switch_root:/# lvm vgscan
    Found volume group "rhel" using metadata type lvm2
switch_root:/# lvm lvscan
    ACTIVE       '/dev/rhel/swap' [2.00 GiB] inherit
    ACTIVE       '/dev/rhel/root' [<17.00 GiB] inherit
```

What this means is that the system LVM is OK. The disk is detected, VG `rhel` is intact, the `root` LVM exists and the 
`swap` is also there.

Now type, in the same shell `cat /proc/cmdline`:

```commandline
switch_root:/# cat /proc/cmdline
BOOT_IMAGE=(hd0,msdos1)/vmlinuz-5.14.0-611.45.1.el9_7.x86_64 root=/dev/mapper/rhel-root \n
ro crashkernel=1G-2G:192M,2G-64G:256M,64G-:512M resume=/dev/mapper/rhel-swap \n
rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rd.break
```
`rd.break` stops the boot procedure before the `root` mount. It will open the `initramfs` shell.

mkdir -p /mnt

mount -o ro /dev/mapper/rhel-root /mnt

ls /mnt -> if you see usr var etc everything is ok

cat /mnt/etc/fstab -> here I see /dev/vg_postgres/lvm_pg_data and other lvm that the systemd will try to mount (the 
lvms do not exist because they were not created) 

We will now edit the /mnt/etc/fstab file. We will comment the following lines:

```terminaloutput
/dev/vg_postgres/lvm_pg_data /opt/postgresql/data xfs defaults 0 0
/dev/vg_postgres/lvm_pg_wal /opt/postgresql/wal xfs defaults 0 0
/dev/vg_postgres/lvm_pg_logs /opt/postgresql/logs xfs defaults 0 0
/dev/vg_postgres/lvm_pg_backups /opt/postgresql/backups xfs defaults 0 0
```

If you cannot save changes, exit the `vi`, and try this:

```commandline
mount | grep mnt
```

If you see `ro` you are in read-only mode. 

You have to :

```commandline
mount -o remount,rw /mnt
```

And then try again to use the `vi`: 

```commandline
vi /mnt/etc/fstab
```

umount -R /mnt

reboot -f


## TO-DO 

Style the steps above and learn and document the process of boot. 

Learn about initramfs and emergency mode. 

Why we need to mount the rhel lvm to /mnt in order to edit /etc/fstab ? 

Reproduce the problem and try to solve it again. Enter in details. Create post on Linkedin with screenshots and try
to explain the solution. 

Try to solve similar boot problems. 


## Unit postgresql-14.service has a bad unit file setting.

### Context 

Date: 29.04.2026

Yesterday I was trying to change the `default location` of the `PGDATA` by following these steps: 
https://fitodic.github.io/how-to-change-postgresql-data-directory-on-linux.

The interesting situation came when I was trying to change the `systemd` configuration for the `postgresql-14.service`, 
by creating a configuration file (.conf) under the `/etc/systemd/system/postgresql-14.service.d/` path.  

The file I created had a bad configuration which resulted in the following problem: 
`Unit postgresql-14.service has a bad unit file setting.` which means that the `systemd` cannot start the 
`postgresql-14.service` using the updated configurations of the service unit file.

To have a better understanding of the situation, this is the output of the 
`sudo systemctl status postgresql-14.service`:

```commandline
[student@localhost ~]$ sudo systemctl status postgresql-14
Warning: The unit file, source configuration file or drop-ins of postgresql-14.service changed on disk. Run 'systemctl daemon-reload' to reload units.
○ postgresql-14.service - PostgreSQL 14 database server
     Loaded: bad-setting (Reason: Unit postgresql-14.service has a bad unit file setting.)
    Drop-In: /etc/systemd/system/postgresql-14.service.d
             └─postgresql-service.conf
     Active: inactive (dead)
       Docs: https://www.postgresql.org/docs/14/static/
             https://www.postgresql.org/docs/14/static/
```

In order to solve this problem, fix the `.conf` file from the `/etc/systemd/system/postgresql-14.service.d/` path and 
run these commands (): 

```commandline
[student@localhost postgresql-14.service.d]$ sudo systemctl daemon-reexec
[student@localhost postgresql-14.service.d]$ sudo systemctl daemon-reload
```

`systemctl daemon-reload`:
- reloads the unit files of the systemd(service, socket, etc.)
- re-reads the `/etc/systemd/system`, `/usr/lib/systemd/system/` and all the `.d` directories 

`systemctl daemon-reexec`
- restarts the systemd process (PID 1), but keeps the running services active
- does not stop the system
