# Disk Mount Guide

Use the steps below to format a new disk, mount it, and make the mount permanent across reboots.

## 1. Identify the disk

First, check your storage drives with `lsblk`:

```bash
badal@homeserver:~$ lsblk
NAME   MAJ:MIN RM    SIZE RO TYPE MOUNTPOINTS
sda      8:0    0  119.2G  0 disk
├─sda1   8:1    0  113.2G  0 part /
├─sda2   8:2    0      1K  0 part
└─sda5   8:5    0    6.1G  0 part [SWAP]
sdb     8:16    0  111.8G  0 disk
└─sdb1  8:17    0  111.8G  0 part
```

In this example, `sda` is the operating system drive and `sdb1` is the storage drive. Replace it with your own disk name if yours is different.

## 2. Format the partition as ext4

If the disk is empty or you want to reformat it, create an ext4 filesystem. If your disk contains files then skip this step:

```bash
sudo mkfs.ext4 /dev/sdb1
```

Example output:

```bash
badal@homeserver:~$ sudo mkfs.ext4 /dev/sdb1
[sudo] password for badal:
mke2fs 1.47.2 (1-Jan-2025)
/dev/sdb1 contains an exfat file system labelled 'Cloud'
Proceed anyway? (y,N) y
Creating filesystem with 29304832 4k blocks and 7331840 inodes
Filesystem UUID: 1681f4f5-3d20-45ed-8e04-f2e0f436625e
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done
Writing inode tables: done
Creating journal (131072 blocks): done
Writing superblocks and filesystem accounting information: done
```

## 3. Create a mount point

Create the folder where the disk will be mounted:

```bash
sudo mkdir -p /mnt/nextcloud-data
```

## 4. Mount the disk

Mount the partition to the folder:

```bash
sudo mount /dev/sdb1 /mnt/nextcloud-data
```

## 5. Get the UUID

To make the mount persistent after a reboot, get the disk UUID:

```bash
sudo blkid /dev/sdb1
```

Example output:

```bash
badal@homeserver:~$ sudo blkid /dev/sdb1
/dev/sdb1: UUID="1681f4f5-3d20-45ed-8e04-f2e0f436625e" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="4c9436a1-58c0-4f2b-8892-0d3bb6b7ac2a"
```

Copy the UUID. You will need it in the next step.

## 6. Add the disk to fstab

Edit `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Add a line like this at the end of the file:

```bash
UUID="1681f4f5-3d20-45ed-8e04-f2e0f436625e" /mnt/nextcloud-data ext4 defaults,nofail 0 0
```

Use `ext4` only if your disk is formatted as ext4. If you used a different filesystem, replace it with the correct type.

Then press `Ctrl+O`, Enter, and `Ctrl+X` to save and exit.

## 7. Verify the mount

Check that the fstab entry works:

```bash
sudo mount -a
```

If no error appears, the mount is configured successfully.

