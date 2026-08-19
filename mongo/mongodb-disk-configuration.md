# MongoDB Disk Configuration Guide

This document describes two common disk layouts for MongoDB on Ubuntu:

1.  **Recommended:** a dedicated data disk for MongoDB.
2.  **Alternative:** MongoDB data stored on the system/root disk.

The examples assume Ubuntu 24.04 and MongoDB running with WiredTiger.

------------------------------------------------------------------------

## 1. Recommended Layout: Dedicated MongoDB Data Disk

### 1.1 Disk layout

Example:

``` text
/dev/vda   50 GiB   ext4   /
/dev/vdb  150 GiB   XFS    /data/mongodb
```

The system disk contains Ubuntu, packages, and operating-system logs.
The dedicated data disk contains MongoDB database files, indexes,
journal files, and WiredTiger data.

Example `lsblk` output before configuring the data disk:

``` text
NAME     SIZE TYPE FSTYPE MOUNTPOINTS
vda       50G disk
├─vda1    49G part ext4   /
├─vda14    4M part
├─vda15  106M part vfat   /boot/efi
└─vda16  913M part ext4   /boot
vdb      150G disk
```

> **Warning:** Verify the device name carefully before formatting.
> `mkfs.xfs` destroys existing data on the selected device. In this
> example, `/dev/vdb` is the new MongoDB data disk.

### 1.2 Install XFS utilities

``` bash
sudo apt update
sudo apt install -y xfsprogs
```

### 1.3 Create an XFS filesystem

For a disk dedicated entirely to MongoDB, the filesystem can be created
directly on the block device:

``` bash
sudo mkfs.xfs /dev/vdb
```

Verify it:

``` bash
sudo blkid /dev/vdb
```

Example:

``` text
/dev/vdb: UUID="96dc0566-a049-4546-943c-c270ff8b4cd8" TYPE="xfs"
```

### 1.4 Create the MongoDB mount point

``` bash
sudo mkdir -p /data/mongodb
```

### 1.5 Mount the data disk

``` bash
sudo mount /dev/vdb /data/mongodb
```

Verify:

``` bash
df -hT /data/mongodb
```

Expected result:

``` text
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/vdb       xfs   150G  ...   ...   ... /data/mongodb
```

### 1.6 Configure persistent mounting

Use the filesystem UUID instead of relying on `/dev/vdb`, because device
names can potentially change.

Get the UUID:

``` bash
sudo blkid /dev/vdb
```

Edit `/etc/fstab`:

``` bash
sudo nano /etc/fstab
```

Add:

``` text
UUID=<VDB_UUID> /data/mongodb xfs defaults,nofail 0 2
```

For example:

``` text
UUID=96dc0566-a049-4546-943c-c270ff8b4cd8 /data/mongodb xfs defaults,nofail 0 2
```

Make sure the mount path is exactly:

``` text
/data/mongodb
```

and not:

``` text
/datamongodb
```

Reload systemd after modifying `fstab`:

``` bash
sudo systemctl daemon-reload
```

Test the configuration before rebooting:

``` bash
sudo umount /data/mongodb
sudo mount -a
```

`mount -a` should complete without errors.

Verify again:

``` bash
df -hT /data/mongodb
lsblk -f
```

Expected layout:

``` text
vda
├─vda1  ext4  /
├─vda15 vfat  /boot/efi
└─vda16 ext4  /boot

vdb     xfs   /data/mongodb
```

### 1.7 Configure ownership after MongoDB installation

After MongoDB has been installed and the `mongodb` user exists:

``` bash
sudo chown -R mongodb:mongodb /data/mongodb
```

Verify:

``` bash
ls -ld /data/mongodb
```

### 1.8 Configure MongoDB to use the dedicated disk

Edit the MongoDB configuration file, typically:

``` bash
sudo nano /etc/mongod.conf
```

Set:

``` yaml
storage:
  dbPath: /data/mongodb
```

Restart MongoDB after completing the remaining MongoDB configuration:

``` bash
sudo systemctl restart mongod
```

Verify the running configuration and disk usage:

``` bash
df -hT /data/mongodb
sudo systemctl status mongod
```

### 1.9 Apply to replica-set nodes

Repeat the disk setup independently on every MongoDB node.

Example:

``` text
mongo-01  10.10.0.11  /dev/vdb -> XFS -> /data/mongodb
mongo-02  10.10.0.12  /dev/vdb -> XFS -> /data/mongodb
mongo-03  10.10.0.13  /dev/vdb -> XFS -> /data/mongodb
```

Each disk will have its **own UUID**. Never copy the `/etc/fstab` UUID
from one server to another.

------------------------------------------------------------------------

## 2. Alternative Layout: MongoDB on the System Disk

A separate data disk is not mandatory. MongoDB can store its data on the
root/system disk when the VM has only one disk.

Example:

``` text
/dev/vda
└─ / (ext4 or XFS)
   ├─ Ubuntu
   ├─ system logs
   └─ MongoDB data
```

This approach is simpler but provides less isolation between the
operating system and database storage.

### 2.1 Option A: Use MongoDB's default data directory

Package-based MongoDB installations commonly use:

``` text
/var/lib/mongodb
```

If this directory is on the root filesystem, no additional disk
formatting or `/etc/fstab` configuration is required.

After installation, verify:

``` bash
df -hT /var/lib/mongodb
```

Example:

``` text
Filesystem      Type  Size  Used Avail Use% Mounted on
/dev/vda1       ext4  200G   ...   ...   ... /
```

MongoDB configuration can remain:

``` yaml
storage:
  dbPath: /var/lib/mongodb
```

Ensure ownership is correct:

``` bash
sudo chown -R mongodb:mongodb /var/lib/mongodb
```

### 2.2 Option B: Use `/data/mongodb` on the root filesystem

If a consistent directory structure is preferred across environments,
create:

``` bash
sudo mkdir -p /data/mongodb
sudo chown -R mongodb:mongodb /data/mongodb
```

Then configure:

``` yaml
storage:
  dbPath: /data/mongodb
```

In this scenario, `/data/mongodb` is only a normal directory on `/`; it
is **not a separate filesystem**.

Verify:

``` bash
df -hT /data/mongodb
```

The output will show the root device, for example:

``` text
Filesystem      Type  Size  Used Avail Use% Mounted on
/dev/vda1       ext4  200G   ...   ...   ... /
```

This is expected for the single-disk layout.

------------------------------------------------------------------------

## 3. Dedicated Data Disk vs. Single Disk

  -----------------------------------------------------------------------------
  Item                    Dedicated Data Disk           System Disk Only
  ----------------------- ----------------------------- -----------------------
  Example                 `/dev/vdb -> /data/mongodb`   `/dev/vda1 -> /`

  MongoDB filesystem      Prefer XFS                    Existing root
                                                        filesystem

  OS/database isolation   Yes                           No

  Separate volume         Easier                        Not possible
  resizing                                              independently

  Separate volume         Easier                        Coupled with OS disk
  snapshot/management                                   

  Database filling OS     Reduced risk                  Higher risk
  filesystem                                            

  Setup complexity        Higher                        Lower

  Recommended for         Yes                           Suitable for
  production                                            simpler/smaller
                                                        deployments
  -----------------------------------------------------------------------------

The main operational benefit of a dedicated disk is **separating
database storage from the operating system**. If MongoDB consumes most
of its data volume, it does not directly consume all free space on the
root filesystem.

With a single disk, monitor free space carefully because MongoDB data,
operating-system files, application packages, and logs all compete for
the same filesystem.

------------------------------------------------------------------------

## 4. Verification Commands

### Dedicated data disk

``` bash
lsblk -f
df -hT /data/mongodb
findmnt /data/mongodb
sudo blkid /dev/vdb
```

`findmnt` should show `/data/mongodb` backed by the dedicated XFS
filesystem.

### System disk only

``` bash
lsblk -f
df -hT /
df -hT /var/lib/mongodb
```

It is normal for `/var/lib/mongodb` or `/data/mongodb` to report the
same root filesystem as `/`.

------------------------------------------------------------------------

## 5. Important Operational Notes

-   Always confirm the target block device before running `mkfs`.
-   Do not format the system disk by mistake.
-   Use the filesystem UUID in `/etc/fstab` rather than copying device
    names between servers.
-   Each replica-set node has a different filesystem UUID.
-   Test `sudo mount -a` before rebooting after changing `/etc/fstab`.
-   Confirm that `/data/mongodb` is actually mounted from the data disk
    before starting MongoDB.
-   After MongoDB is installed, ensure the MongoDB service account owns
    the database directory.
-   Monitor both free disk space and disk latency/IOPS in production.
-   A dedicated data volume does not replace backups. Replica sets
    provide availability, not a backup strategy.
