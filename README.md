# Safe

The 'Safe' script facilitates easy access to open a filesystem stored in a LUKS encrypted data file.

By storing confidential data within an encrypted filesystem in this manner the data is protected from users and programs that have access to your system.

Even if someone or something has accessed to the overall encrypted filesystem on your computer, the data within this encrypted file is inaccessible while locked.

Users use 'safe' to open the encrypted filesystem, access data, then close it.  This ensures that the data is protected most of the time.

## Operation

### Safe open

```bash
$ sudo safe open my-safe

Enter passphrase for /home/maxwell/Documents/Safes/my-safe.luks:
/dev/mapper/LUKS_my-safe on /mnt/secure/my-safe type ext4 (rw,relatime,seclabel)

Your files are now available at /mnt/secure/my-safe
REMEMBER TO CLOSE IT with: 'safe close my-save'
```

This command makes the contents of an encrypted safe file accessible to the user as a mounted filesystem.  cryptsetup is used to open access to the safe, then the filesystem within the safe is mounted at a location such as /mnt/secure/my-save.

Users use any program, script or command line to access the data.

### Safe close

```bash
$ sudo safe close my-safe

Mount check after unmounting:
Safe       Open User       Map Name        Mount point             Filename       
----       ---- ----       --------        -----------             --------       
my-safe    no   maxwell    LUKS_my-safe    /mnt/secure/my-safe     my-safe.luks     

-rw-r--r--. 1 maxwell maxwell 1.0G Dec 24 22:51 /home/maxwell/Documents/Safes/my-safe.luks

Cryptsetup status:
/dev/mapper/LUKS_my-safe is inactive.
```

Immediately upon finishing with access to the data, users use this command to unmount the filesystem and close access to the safe's encrypted filesystem.  The data is now secure.

### Safe status
```bash
$ sudo safe status my-safe

Safe       Open User       Map Name        Mount point             Filename       
----       ---- ----       --------        -----------             --------       
my-safe    yes  maxwell    LUKS_my-safe    /mnt/secure/my-safe     my-safe.luks     

-rw-r--r--. 1 maxwell maxwell 1.0G Dec 24 23:09 /home/maxwell/Documents/Safes/my-safe.luks

Cryptsetup status:
/dev/mapper/LUKS_my-safe is active and is in use.
  type:    LUKS1
  cipher:  aes-xts-plain64
  keysize: 256 bits
  key location: dm-crypt
  device:  /dev/loop0
  loop:    /home/maxwell/Documents/Safes/my-safe.luks
  sector size:  512
  offset:  4096 sectors
  size:    2093056 sectors
  mode:    read/write
```

This command shows the current status (open or closed) of a configured safe file.

### Safe list

```bash
$ sudo safe list
Safe       Open User       Map Name        Mount point             Filename       
----       ---- ----       --------        -----------             --------       
accts      no   maxwell    LUKS_accts      /mnt/secure/accts       accts.luks     
receipts   no   maxwell    LUKS_receipts   /mnt/secure/receipts    receipts.luks     
travel     no   maxwell    LUKS_travel     /mnt/secure/travel      travel.luks
```

This command lists safes configured in a configuration file, /root/.safe.conf

# Techincal Details

Safe is a wrapper around 'cryptsetup' which handles opening and closing access to encrypted data.

Data is stored in layers which visually look like this:

Layer | Description
----- | -----------
Layer 3| Private User Data
Layer 2| Formatted ext4 filesystem
Layer 1| LUKS encrypted data
Layer 0| Normal Linux file

## Config File $HOME/.safe.conf

A configuration file, $HOME/.safe.conf, defines a directory where safes are stored as well as declarations for each safe file:

**safe_name** - A unique name for each Safe

**safe_user** - The user name of the owner of this data

**safe_mapper** - A unique name device mapper will provide in /dev/mapper for the accessible unencrypted data.  This will act like a normal block device for formatting and mounting.

**safe_mount** - Where the formatted filesystem should be mounted

**safe_filename** - The filename of the safe in the safe directory
```
#
# Personal config file for 'safe' script to lock/unlock encrypted filesystems
#

safe_directory /home/maxwell/Documents/Safes

# Syntax:
#safe_file <safe_name> <safe_user> <safe_mapper> <save_mount> <safe_filename>

# Accounting Files
safe_file accts maxwell LUKS_accts /mnt/secure/accts accts.luks

# Legal information
safe_file receipts maxwell LUKS_receipts /mnt/secure/receipts receipts.luks

# Personal Information
safe_file travel maxwell LUKS_travel /mnt/secure/travel travel.luks
```

## Requirements

Users must use 'sudo' to call this script because cryptsetup will use the Linux device mapper facility to make the encrypted data appear as a block device.

### Sparse or normal files

When preparing a data file to store an encrypted and embedded Linux filesystem, you can choose to use 'sparse' files or 'normal' files.

A 'sparse' file pre-allocates a set amount of space without actually consuming the space on disk. This saves actual disk blocks, limits writes to create the initial file to prep time is reduced and may have other benefits.

A 'normal' file allocates a set amount of space and consumes that amount of space immediately.  The main advantage of this is that a file requiring 4G of space will have it allocated immediately and never run into problems attempting to expand later if the host filesystem is full.

When using files for safes, I often use a large sparse file such as 1G or 4G in size, then only store some number of megabytes initially.

If you want maximum security, you will write all zeros to the file using the command suggested below.  This will allocate 100% of the space in a sparse file, so it becomes equivalent to a normal file at this point.

#### PREPARATION BEFORE USING THIS SCRIPT:

1. Create an empty file; This example makes a 1G file.
```
# Create a 1G sparse file
truncate -s 1G my-safe-file.luks

# OR, the old fashioned way:

# Sparse
dd if=/dev/zero of=my-sparse-safe.luks bs=1G count=0 seek=1

# Normal
dd if=/dev/zero of=my-normal-safe.luks bs=1G count=1
```

1. Encrypt the data with luks
```
cryptsetup luksFormat my-safe.luks
```

1. Open the encrypted data, mapping it as a block device using device mapper
```
cryptsetup luksOpen my-safe.luks my-safe
```

1. Format the LUKS data space.  This ensures that the data within the filesystem is randomized.
```
pv -tpreb /dev/zero | dd of=/dev/mapper/my-safe bs=1M
```
1. Format the mapped block device. You can use XFS or any other valid filesystem.
```
mkfs.ext4 -v -m 0 -j -L my-safe /dev/mapper/my-safe
```
1. Mount the newly created filesystem
```
mkdir /mnt/my-open-luks
mount /dev/mapper/my-open-luks /mnt/my-open-luks
```

1. Add a declaration to the configuration file for this new safe file:

```
# Bank Statements
safe_file bank maxwell LUKS_bank /mnt/secure/bank bank.luks
```
