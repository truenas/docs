External SMB shares are essentially redirects to shares on other systems.
Administrators might want to use this when managing multiple TrueNAS systems with SMB shares, and if they do not want to keep track of which shares are on which boxes for clients.
This feature allows admins to see and connect to any TrueNAS system with external shares active.

Create the SMB share on another TrueNAS remote server (for example, *system1*). See [Adding and Managing SMB Shares](/shares/smb/adding-and-managing-smb-shares) for instructions on creating SMB shares.

We recommend using Active Directory or LDAP when creating user accounts, but at a minimum, synchronize user accounts between the system with the share (*system1*) and on the TrueNAS system where you set up the external share (for example, *system2*).

On *system2* (the local system), select **External Share**, enter the full domain name or IP address, and the share name.
Separate the server and share name with the `\` character. Example: *192.168.0.200\SHARE* in **Remote Path**.

Click **Save** to add the share.

Repeat the *system2* instructions above on *system1* to see the SMB shares on each system.

{{< trueimage src="/images/shares/adding-an-external-share.png" alt="Set Up Another External SMB Share" id="Set Up Another External SMB Share" >}}

Repeat for each TrueNAS system with SMB shares to add as an external share.

### Setting Up an External Share with an Earlier Release

When setting up an external share between TrueNAS systems that are on different releases, for example, one system is on 25.04 and the other is on the latest release of 25.10, follow the external share instructions for each release.

Set the TrueNAS 25.04 system SMB **Purpose** to the default preset, leave the default settings associated with this share as is, and then enter the redirect path to share on the 25.10 system as **EXTERNAL:*ipaddress\sharename*** in the **Path** field. For example, *EXTERNAL:10.220.3.33\testshare2*.
Be aware, changing the path also changes the SMB share name. Verify the share name is set to the desired or existing share name and not renamed to the redirect string in **Path**.

{{< trueimage src="/images/shares/set-up-external-smb-share.png" alt="Set Up Another External SMB Share" id="Set Up Another External SMB Share" >}}

Set the TrueNAS 25.10 system SMB **Purpose** to **External Share**, and then enter the path to the share on the 25.04 system as *ipaddress*\*sharename* in the **Remote Path** field. For example, *10.220.1.34*\*testshare*.

{{< trueimage src="/images/shares/adding-an-external-share.png" alt="Set Up Another External SMB Share" id="Set Up Another External SMB Share" >}}

Add descriptions to each share that identify the purpose of the share.
The description shows on the **Windows (SMB) Shares** widget and the **SMB** screen.

**Save** changes made to the share.
