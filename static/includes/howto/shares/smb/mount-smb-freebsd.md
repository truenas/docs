Mounting on a FreeBSD system involves creating the mount point and mounting the volume.

Create a mount point using the `sudo mkdir /mnt/smb_share` command.

Mount the volume using the `sudo mount_smbfs -I computer_name\share_name /mnt/smb_share` command.
