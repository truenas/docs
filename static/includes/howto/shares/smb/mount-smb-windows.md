To permanently mount the SMB share in Windows, map a drive letter in the computer for the user to the TrueNAS IP and share name.
Select a drive letter from the bottom of the alphabet rather than from the top to avoid assigning a drive dedicated to some other device.
The example below uses *Z*.
Open the command line and run the following command with the appropriate drive letter, TrueNAS system name or IP address, and the share name.

<code>net use <i>Z</i>: &bsol;&bsol;<i>TrueNAS_name</i>&bsol;<i>share_name</i> /PERSISTENT:YES</code>

Where:

* *Z* is the drive letter to map to TrueNAS and the share
* *TrueNAS_name* is either the host name or the system IP address
* *share_name* is the name given to the SMB share

To temporarily connect to a share, open a Windows File Explorer window, type <code>&bsol;&bsol;<i>TrueNAS_name</i>&bsol;<i>share_name</i></code> and then enter the user credentials to authenticate with to connect to the share.
Windows remembers the user credentials, so each time you connect, it uses the same authentication credentials unless you restart the system.
After restarting, you are prompted to enter the authentication credentials again.
