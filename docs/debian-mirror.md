# About

The Debian mirror project aims to host the two most recent versions of Debian releases and their package repositories in Mauritius.

## Preparing the disk

We use LVM to manage the disks. It ensures future re-provisioning to be smooth. Debian does not have LVM installed by default.

```
apt install lvm2
```

Create an LVM physical volume on top of `/dev/vdb`.

```
pvcreate /dev/vdb
```

Next, create a volume group called `vg-mirorr` using `/dev/vdb`.

```
vgcreate vg-mirror /dev/vdb
```

You can view the details of the physical volume and volume group using the `pvdisplay /dev/vdb` and `vgdisplay vg-mirror` respectively.

Use all the available space to create a logical volume named `vol-mirror`.

```
lvcreate -n vol-mirror -l 100%FREE vg-mirror
```

Use `lvdisplay` to see details of the logical volume.

```
lvdisplay vg-mirror/vol-mirror
```

Create a file system on top of the newly created logical volume.

```
mkfs.ext4 /dev/vg-mirror/vol-mirror
```

Use the `blkid` command to obtain the UUID of the volume.

```
blkid /dev/vg-mirror/vol-mirror
```

Create the `/vol-mirror` directory and add entry for the volume in `/etc/fstab`, e.g:

```
UUID=e1929239-5087-44b1-9396-53e09db6eb9e      /vol-mirror    ext4    defaults    0 0
```

Mount the volume.

```
mount /vol-mirror
```
