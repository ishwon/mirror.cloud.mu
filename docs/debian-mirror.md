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

Next, create a volume group called `vg-mirror` using `/dev/vdb`.

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

## Setup the Mirror

Install `debmirror` and its dependency `gnupg`.
```
apt-get install debmirror gnupg
```

Import gpg keys. Debmirror uses gpg to verify Release and Release.gpg using the default keying ~/.gnupg/trustedkeys.gpg.
This can be altered by exporting the environmental variable `GNUPGHOME`.
```
gpg --no-default-keyring --keyring trustedkeys.gpg --import /usr/share/keyrings/debian-archive-keyring.gpg
```

Create the mirror directory and its subdirectories. It should look like this:
```
/vol-mirror/debian-mirror/
├── dists
├── pool
└── project
```

Copy the configuration file example to `~/.debmirror.conf` or `/etc/debmirror` and edit it.
Example of the configuration file is located at `/usr/share/doc/debmirror/examples`

```
cp /usr/share/doc/debmirror/examples/debmirror.conf /etc/debmirror.conf
```

Content of debmirror config file:
```
# Default config for debmirror

# The config file is a perl script so take care to follow perl syntax.
# Any setting in /etc/debmirror.conf overrides these defaults and
# ~/.debmirror.conf overrides those again. Take only what you need.
#
# The syntax is the same as on the command line and variable names
# loosely match option names. If you don't recognize something here
# then just stick to the command line.
#
# Options specified on the command line override settings in the config
# files.

# Location of the local mirror (use with care)
# $mirrordir="/path/to/mirrordir"
$mirrordir="/vol-mirror/debian-mirror";

# Output options
$verbose=0;
$progress=0;
$debug=0;

# Download options
# host is Kenya
$host="debian.mirror.liquidtelecom.com";
$user="anonymous";
$passwd="anonymous@";
$remoteroot="debian";
$download_method="rsync";
@dists="stable,bullseye-updates,bullseye-backports";
@sections="main,main/debian-installer,contrib,non-free,non-free-firmware";
@arches="amd64";
# @ignores="";
# @excludes="";
# @includes="";
# @excludes_deb_section="";
# @limit_priority="";
$omit_suite_symlinks=0;
$skippackages=0;
# @rsync_extra="doc,tools";
$i18n=0;
$getcontents=0;
$do_source=1;
$max_batch=0;

# @di_dists="dists";
# @di_archs="arches";

# Save mirror state between runs; value sets validity of cache in days
$state_cache_days=0;

# Security/Sanity options
$ignore_release_gpg=0;
$ignore_release=0;
$check_md5sums=0;
$ignore_small_errors=0;

# Cleanup
$cleanup=0;
$post_cleanup=1;

# Locking options
$timeout=300;

# Rsync options
$rsync_batch=200;
$rsync_options="-aIL --partial";

# FTP/HTTP options
$passive=0;
# $proxy="http://proxy:port/";

# Dry run
$dry_run=0;

# Don't keep diff files but use them
$diff_mode="use";

# The config file must return true or perl complains.
# Always copy this.
1;
```

Consult the manpage for the options as well as the [Debian wiki](https://www.debian.org/mirror/ftpmirror). Here is the [list](https://www.debian.org/mirror/list-full) of public mirrors which support rsync and that are good to mirror from.

Launch `debmirror`, in a `byobu` or `tmux` session, to start the synchronization process. `debmirror` can be launched without any options as it reads the configuration file by defaults.

Add a cronjob to update the mirror after a few hours.
```
0 */6 * * * /usr/bin/debmirror
```

