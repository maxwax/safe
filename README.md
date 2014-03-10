#### SUMMARY

The goal of this program is to keep sensitive data locked and secure
at any time when it is not directly being used by the user.

The user uses 'safe' to unlock a LUKS encrypted data file and have
an embedded filesystem mounted.  The data is then available to the user
similar to a USB storage disk being mounted.  When the user is done 
accessing the data it is unmounted and locked again.

By keeping data secure when not directly in use, it limits the ability
for intruders and other unauthorized users to gain access to it. If an
intruder gains access to your system in the middle of the night, they
will download encrypted files instead of raw files.

While this doesn't completely protects your data, it is an improvement 
over raw data sitting for the taking..

#### HOW IT WORKS

Data is stored in layers which visually look like this:

Layer 3: User data as files

Layer 2: ext4 filesystem

Layer 1: LUKS encrypted data

Layer 0: Simple Linux file

Using the 'open' command, this script points to the Linux file and
unlocks its LUKS-secured data, revealing an ext4 filesystem.  The ext4
filesystem is then mounted and the user can access files with normal
tools.

Using the 'close' command, this script unmounts the mounted filesystem
then closes the mapped LUKS data, leaving the file encrypted and
protected from others.

Ideally, the secure data is opened only as needed and quickly closed
afterwards.  This way, if a system intrusion takes place, the attacker
will only gain access to encrypted data and not valuable information.

#### PREPARATION BEFORE USING THIS SCRIPT:

Create an empty file; This example makes a 32M file.
```
dd if=/dev/zero of=my-luks-locker.luks bs=1M count=32
```

Encrypt the data with luks
```
cryptsetup luksFormat my-luks-locker.luks
```

Open the encrypted data, mapping it as a block device using device mapper
```
cryptsetup luksOpen my-luks-locker.luks my-open-luks
```

Format the mapped block device
```
mkfs.ext4 -v -m 0 -j -L my-luks-locker /dev/mapper/my-open-luks
```

Mount the newly created filesystem
```
mkdir /mnt/my-open-luks
mount /dev/mapper/my-open-luks /mnt/my-open-luks
```

Modify this script to add a case statement for each luks encrypted
file:

```
USER="maxwell"
SOURCE_FILE="/home/maxwell/Documents/Projects/Notes/notes.luks"
UNLOCKED_MAPPING="notes-luks"
MOUNT_POINT="/mnt/secure/notes";;
```

This part will be replaced by an external config file in the future.

#### USAGE

Open a safe:
```
linux # safe open notes

Enter passphrase for /home/maxwell/Documents/Safes/notes.luks: 
Your files are now available at /mnt/secure/notes
Issue safe-close notes to close it

```

Open all known safes:
```
# safe open all
```

Close a safe:
```
# safe close notes

The safe notes is now closed and locked.
```

Close all safes:
```
# safe close all
```

Show available safes:
```
# safe list
```

Status a safe:
```
# safe status notes

== Safe 'notes' ==
/dev/mapper/notes-luks is active and is in use.
  type:    LUKS1
  cipher:  aes-xts-plain64
  keysize: 4096 bits
  device:  /dev/loop0
  loop:    /home/maxwell/Documents/Safes/notes.luks
  offset:  4096 sectors
  size:    61440 sectors
  mode:    read/write
/dev/mapper/notes-luks on /mnt/secure/notes type ext4 (rw,relatime,seclabel,data=ordered)

```

Show the status of all safes:
```
# safe status all
```
