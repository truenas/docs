---
title: "Adding and Managing SMB Shares"
description: "Complete workflow for creating SMB shares, configuring users, tuning permissions, and managing existing shares."
weight: 15
doctype: tutorial
tags:
- smb
- shares
- acl
---

{{< include file="/static/includes/root-level-dataset-share-warning.md" >}}

{{< hint type="info" title="Active Directory and SMB Service" >}}
Verify your Active Directory connections are working and error-free before adding an SMB share.
When an SMB share is configured but not working or is in an error state, AD cannot bind, and TrueNAS cannot start the SMB service.
{{< /hint >}}

Creating an SMB share on your system requires adding the share and then getting it working.

TrueNAS uses [Samba](https://www.samba.org/) to provide SMB services.
The SMB protocol has multiple versions. During the SMB session negotiation, a typical SMB client can negotiate the highest supported SMB protocol.
Industry-wide, SMB1 protocol (sometimes referred to as NT1) use is deprecated for security reasons.

{{< include file="/static/includes/smb-share-msdos-alert.md" >}}

However, most SMB clients support SMB 2 or 3 protocols even when they are not the default.

{{< hint type=note >}}
Legacy SMB clients rely on NetBIOS name resolution to discover SMB servers on a network.
TrueNAS disables the NetBIOS name server (nmbd) by default. Enable it on the **Network > Global Settings** screen if this functionality is required.

Mac OS clients use mDNS to discover SMB servers present on the network. TrueNAS enables the mDNS server (avahi) by default.

Windows clients use [WS-Discovery](https://docs.oasis-open.org/ws-dd/ns/discovery/2009/01) to discover the presence of an SMB server.
You can disable network discovery by default depending on the Windows client version.

Discoverability through broadcast protocols is a convenience feature and is not required to access an SMB server.
{{< /hint >}}

## Sharing Administrator Access

{{< include file="/static/includes/sharing-admin-role.md" >}}

## Creating SMB Share User Accounts

You can manually add user accounts or use directory services like Active Directory or LDAP to provide additional user accounts.
If setting up an external SMB share, we recommend using Active Directory or LDAP, or at a minimum, synchronizing the user accounts between systems.

{{< include file="/static/includes/howto/shares/smb/create-smb-user-accounts.md" >}}

## Adding an SMB Share and Dataset

{{< include file="/static/includes/howto/shares/smb/add-smb-share-basic.md" >}}

## Configuring Advanced Options

A basic SMB share does not need to use the **Advanced Options** settings. Click **Advanced Options** to finish customizing the SMB share settings.

See [SMB Shares Screens](# <!-- TODO-REF: SMBSharesScreens -->) for all settings and other possible use cases.

{{< expand "Guest Access" "v" >}}
{{< hint type=warning >}}
Guest access adds security vulnerabilities and should be avoided.
{{< /hint >}}

Guest access allows users to connect to an SMB share without providing credentials.
In TrueNAS SCALE 25.10 and later, this feature is only available for shares with the **Legacy Share** preset (shares that used **No Preset** in releases before 25.10).

To enable guest access on a **Legacy Share**:

1. Go to **Shares** and click on the share name to expand the share widget, then click **Edit**.
2. Click **Advanced Options** and scroll down to the **Access** settings.
3. Select the **Allow Guest Access** checkbox.
4. Click **Save**.

The privileges granted are the same as those for a guest account.

{{< hint type=warning >}}
Windows 10 version 1709 and later--and Windows Server 2019 and later--disable guest access by default as a security measure.
Windows clients require additional configuration to connect to shares with guest access enabled.
To enable guest access on Windows clients, modify Windows registry settings or Group Policy to allow insecure guest logons.
See Microsoft documentation for configuration details.
{{< /hint >}}

For new shares:

New shares created in TrueNAS SCALE 25.10 and later cannot select the **Legacy Share** preset, and therefore cannot use guest access.
Guest access has been limited to legacy shares due to security concerns and client-side deprecation:

* Windows 10 version 1709 and Windows Server 2019 and later disable guest access by default
* Major SMB client vendors are deprecating guest users
* Guest sessions cannot use signing and encryption features

Client-specific behavior:

* Mac OS clients - Prevent automatic connection as guest account.
  Users must explicitly select **Connect As: Guest**.
  See the [Apple documentation](https://support.apple.com/guide/mac-help/connect-mac-shared-computers-servers-mchlp1140/mac) for more details.
* Windows clients - Require registry or Group Policy modifications to enable insecure guest authentication

Alternative approaches:

Instead of guest access, consider these secure alternatives:

* Create a dedicated guest user account with limited permissions
* Use share ACL to restrict the guest user to read-only access

If the share is nested under parent datasets, see [Using the Traverse Permission](#using-the-traverse-permission).
{{< /expand >}}

{{< expand "Read or Write Access" "v" >}}
To prohibit writes to the share, select **Export Read-Only**.

Select **Access Based Share Enumeration** to restrict share visibility for users with read or write access to the share.
This setting applies to datasets with a POSIX ACL type.
For datasets with NFSv4 ACL type, access-based enumeration is automatically enabled and does not allow disabling.
See the [smb.conf](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html) manual page.
{{< /expand >}}

{{< expand "Host Allow and Host Deny" "v" >}}
{{< hint type=note >}}
Hosts Allow and Hosts Deny settings are available for all share presets except **External Share**.
{{< /hint >}}

Use the **Host Allow** and **Host Deny** options to allow or deny specific host names and IP addresses.

Use the **Hosts Allow** field to enter a list of allowed IP addresses.
Separate entries by pressing <kbd>Enter</kbd>.
{{< hint type="Warning" title="Setting Host Allow" >}}
Entering values in the **Host Allow** restricts access to only the addresses entered into this list!
This list can break UI access for all other IP or host name entries.
{{< /hint >}}
You can find a more detailed description with examples [here](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html#HOSTSALLOW).
Use the **Hosts Deny** field to enter a list of denied host names or IP addresses. Separate entries by pressing <kbd>Enter</kbd>.

**Hosts Allow** and **Hosts Deny** work together to produce different situations:

* Leaving both **Hosts Allow** and **Hosts Deny** free of entries allows any host to access the SMB share.
* Adding entries to the **Hosts Allow** list but not the **Hosts Deny** list allows only the hosts on the **Hosts Allow** list to access the share.
* Adding entries to the **Hosts Deny** list but not the **Hosts Allow** list allows all hosts not on the **Hosts Deny** list to access the share.
* Adding entries to both a **Hosts Allow** and **Hosts Deny** list allows all hosts on the **Hosts Allow** list to access the share and allows hosts not on the **Hosts Allow** or **Hosts Deny** list to access the share.
{{< /expand >}}

{{< expand "Legacy Share Preset (Upgraded Shares Only)" "v" >}}

When you upgrade to TrueNAS SCALE 25.10 from an earlier release, existing shares that used the **No Preset** option are automatically migrated to the **Legacy Share** preset.
This preset provides access to configuration options that are no longer available for new shares.

Why can't I create a new legacy share?

The **Add SMB** screen does not include **Legacy Share** as an option. This preset only appears in the **Edit SMB** screen for shares created before 25.10.
TrueNAS removed these options from new shares due to:

* Security concerns (guest access, recycle bin)
* Better alternatives available (ZFS snapshots instead of recycle bin)
* Client-side deprecation (guest access no longer supported by major vendors)

Legacy share options:

Legacy shares provide access to additional settings not available in modern presets.

In the **Access** section:

* **Enable ACL** - Configure additional ACL entries for custom access controls
* **Allow Guest Access** - Enable anonymous access without credentials (not recommended)

In the **Other Options** section:

* **Use as Home Share** - Configure share as user home directories (see [Private Dataset Share](# <!-- TODO-REF: smb-private-dataset-share -->) for modern alternative)
* **Time Machine Quota** - Set maximum limit on Time Machine backup storage
* **Legacy AFP Compatibility** - Support for migrated AFP shares
* **Enable Shadow Copies** - Export ZFS snapshots as volume shadow copies for VSS clients (see [Shadow Copies](# <!-- TODO-REF: add-smb-shadow-copies -->))
* **Export Recycle Bin** - Move deleted files to a `.recycle` directory (not recommended - use ZFS snapshots instead)
* **Use Apple-style Character Encoding** - Translate NTFS illegal characters to Unicode private range
* **Enable Alternate Data Streams** - Support multiple NTFS data streams
* **Enable SMB2/3 Durable Handles** - Allow file handles to survive disconnections
* **Enable FSRVP** - Remote VSS protocol support for snapshot management
* **Path Suffix** - Append per-user/computer/IP suffixes to connection path
* **Additional Parameters String** - Add custom smb.conf parameters (advanced users only)
* **VUID** - Time Machine volume UUID for mDNS advertisements

See [Legacy Share Settings](# <!-- TODO-REF: SMBSharesScreens#legacy-share-settings -->) in the UI reference for complete details on each option.
{{< /expand >}}

{{< expand "Apple Filing Protocol (AFP) Compatibility" "v" >}}
AFP shares are deprecated and not available in TrueNAS.

To customize your SMB share to work with a migrated AFP share or with your Mac OS, use the share option on the **Purpose** dropdown list and the **Advanced Options** settings provided for these use cases:

* **Time Machine Share** enables [Apple Time Machine](https://support.apple.com/en-us/HT201250) backups on this share.

**Use Apple-style Character Encoding**, listed under **Other Options** for all share types except **Time Machine Share** and **External Share**, converts NTFS illegal characters like the Mac OS SMB clients do.
By default, Samba uses a hashing algorithm for NTFS illegal characters.
{{< /expand >}}

{{< expand "Private SMB Datasets and Shares" "v" >}}
Used to set up an alternative to the legacy home shares function, select **Private Dataset Share** on the **Purpose** dropdown list, and customize settings listed under **Other Options**.

This allows you to add private datasets and shares for individual users, and is an alternate way to create home shares for them.
See [Setting Up SMB Home Shares](# <!-- TODO-REF: smb-private-dataset-share -->) for more information.
{{< /expand >}}

{{< expand "SMB Audit Logging" "v" >}}
{{< include file="/static/includes/configure-smb-share-auditing.md" >}}
{{< /expand >}}

## Tuning ACLs for SMB Shares

{{< include file="/static/includes/howto/shares/smb/tune-smb-acls.md" >}}

## Starting the SMB Service

{{< include file="/static/includes/howto/shares/smb/start-smb-service.md" >}}

## Mounting the SMB Share

The instructions in this section cover mounting the SMB share on a system with the following operating systems.

{{< expand "Mounting on a Linux System" "v" >}}
{{< include file="/static/includes/howto/shares/smb/mount-smb-linux.md" >}}
{{< /expand >}}

{{< expand "Mounting on a Windows System" "V" >}}
{{< include file="/static/includes/howto/shares/smb/mount-smb-windows.md" >}}
{{< /expand >}}

{{< expand "Mounting on an Apple System" "v" >}}
{{< include file="/static/includes/howto/shares/smb/mount-smb-apple.md" >}}
{{< /expand >}}

{{< expand "Mounting on a FreeBSD System" "v" >}}
{{< include file="/static/includes/howto/shares/smb/mount-smb-freebsd.md" >}}
{{< /expand >}}

## Managing Existing Shares

To access SMB share management options, go to **Shares** and locate the **Windows (SMB) Shares** widget.
The widget lists configured SMB shares, but it is not the complete list.
To see a complete list of shares, click on **Windows (SMB) Shares <span class="material-icons">launch</span>** header to open the **Shares > SMB** screen.
The <span class="material-icons">more_vert</span> dropdown list to the right of each share shows four options that open other screens or dialogs that provide access to share settings.

To manage an SMB share, click <span class="material-icons">more_vert</span> dropdown list to the right of each share to see the options for the share you want to manage. Options are:

* **Edit** opens the **Edit SMB** screen where you can change settings for the share.
* **Edit Share ACL** opens the **Share ACL** screen, where you can [add or edit ACL entries](#configuring-the-smb-share-acl).
* **Edit Filesystem ACL** opens the **Edit ACL** screen, where you can edit the dataset permissions for the share.
  The **Dataset Preset** option determines the ACL type and the type of **ACL Editor** screen that opens (POSIX or NSFv4).
* **Delete** opens a delete confirmation dialog. Use this to delete the share and remove it from the system. Delete does not affect shared data.

## Configuring SMB Auditing

{{< include file="/static/includes/configure-smb-share-auditing.md" >}}

## Modifying ACL Permissions for SMB Shares

{{< include file="/static/includes/share-acl-dialogs.md" >}}

You have two options that modify ACL permissions for SMB shares:
* **Edit Share ACL** modifies ACL permissions that apply to the SMB share.
* **Edit Filesystem ACL** modifies ACL permissions at the share dataset level.

See the [ACL Primer](https://www.truenas.com/docs/references/aclprimer/) for general information on Access Control Lists (ACLs) in general, the [Permissions](# <!-- TODO-REF: PermissionsSCALE -->) article for more details on configuring ACLs, and [**Edit ACL** Screen](# <!-- TODO-REF: EditACLScreens -->) for more information on the dataset ACL editor screens and setting options.

### Configuring the SMB Share ACL

{{< include file="/static/includes/share-acl-permissions.md" >}}

### Configuring Dataset File System ACL

{{< include file="/static/includes/filesystem-acl-permissions.md" >}}

#### Changing the built-in-user Group Permissions

{{< include file="/static/includes/change-builtin-user-acl.md" >}}

#### Adding a New Share Group

{{< include file="/static/includes/add-new-smb-share-group-and-ace.md" >}}

#### Using the Traverse Permission

{{< include file="/static/includes/using-traverse-permission.md" >}}
