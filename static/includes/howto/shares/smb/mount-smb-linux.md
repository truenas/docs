Verify that your Linux distribution has the required CIFS packages installed.

Create a mount point with the `sudo mkdir /mnt/smb_share` command.

Mount the volume with the `sudo mount -t cifs //computer_name/share_name /mnt/smb_share` command.

If your share requires user credentials, add the switch `-o username=` with the username after `cifs` and before the share address.
