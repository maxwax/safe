SUMMARY
=======

*safe* is a wrapper around 'cryptsetup' and 'mount' which make opening
and accessing, then later closing and protecting secure information
contained in regular Linux files.

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

PREPARATION BEFORE USING THIS SCRIPT:
=====================================

Create an empty file; This example makes a 32M file.
    dd if=/dev/zero of=my-luks-locker.luks bs=1M count=32

Encrypt the data with luks
    cryptsetup luksFormat my-luks-locker.luks

Open the encrypted data, mapping it as a block device using device mapper
    cryptsetup luksOpen my-luks-locker.luks my-open-luks

Format the mapped block device
    mkfs.ext4 -v -m 0 -j -L my-luks-locker /dev/mapper/my-open-luks

Mount the newly created filesystem
    mkdir /mnt/my-open-luks
    mount /dev/mapper/my-open-luks /mnt/my-open-luks

Modify this script to add a case statement for each luks encrypted
file:

    USER="maxwell"
    SOURCE_FILE="/home/maxwell/Documents/Projects/Notes/notes.luks"
    UNLOCKED_MAPPING="notes-luks"
    MOUNT_POINT="/mnt/secure/notes";;

This part will be replaced by an external config file in the future.

USAGE
=====

Open a safe:
    safe open notes

Open all known safes:
    safe open all

Close a safe:
    safe close notes

Close all safes:
    safe close all

Show available safes:
    safe list

Status a safe:
    safe status notes

Show the status of all safes:
    safe status
