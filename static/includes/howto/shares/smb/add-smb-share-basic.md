You can create an SMB share while [creating a dataset on the **Add Dataset** screen](# <!-- TODO-REF: DatasetsSCALE -->) or create a dataset and the share using the **Add SMB** share screen.
This article covers adding the dataset using the **Add SMB** share screen.

{{< include file="/static/includes/apps-smb-error-warning.md" >}}

{{< include file="/static/includes/share-datasets-not-pools.md" >}}

If you want to organize the SMB share dataset under a parent dataset (for example, under *smb-shares*), create that parent dataset so you can select it as the parent in step 2 below.
Alternatively, you can create the parent and SMB share dataset using the **Create Dataset** option associated with the file browser in the **Add SMB** screen by making the create dataset instructions a two-step process.

To create a basic Windows SMB share and a dataset, go to **Shares**, then click **Add** on the **Windows Shares (SMB)** widget to open the **Add Share** screen.

{{< trueimage src="/images/shares/add-share-basic-options.png" alt="Add SMB Basic Options" id="Add SMB Basic Options" >}}

1. Enter or browse to select the SMB share mount path (parent dataset where you want to add a dataset for this share).
   You cannot use a root dataset for a share. When the dataset selected has an existing ACL, a warning dialog shows. Click **Continue**.
   Click on the dataset under which you want to add the SMB share dataset.
   The blank **Path** field populates with the path selected in the file browser field directly below it.
   The **Path** file browser field is the directory tree on the local file system that TrueNAS exports over the SMB protocol.

   {{< include file="/static/includes/file-explorer-folder-icons.md" >}}

2. Click **Create Dataset**.
   Enter a name for the dataset in the **Create Dataset** dialog, then click **Create**.
   The system creates the new dataset and populates the **Name** field with the dataset name, which becomes the share name.

   To make the new dataset the parent for an SMB share, select the just-added dataset, then click **Create Dataset** again to add the child dataset for the share.

   The path forms part of the share pathname when SMB clients perform an SMB tree connect.
   Because of how the SMB protocol uses the name, it must be less than or equal to 80 characters.
   Do not use invalid characters as specified in Microsoft documentation MS-FSCC section 2.1.6.

   If you change the name, follow the naming conventions for:
   * [Files and directories](https://learn.microsoft.com/en-us/windows/win32/fileio/naming-a-file#naming-conventions)
   * [Share names](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-fscc/dc9978d7-6299-4c5a-a22d-a039cdc716ea)

3. Select a share type on the **Purpose** dropdown list.
   The share type selected locks or unlocks the pre-determined **Advanced Options** settings for the share.

   Select **Default Share** to create a basic SMB share with the **Browsable to Network Clients** option preselected.
   This determines whether this share name is included when browsing shares.

   Select **Private Datasets Share** to create an alternative to home shares.
   See [Setting Up SMB Home Shares](# <!-- TODO-REF: smb-private-dataset-share -->) for more information on replacing this legacy feature with private SMB shares and datasets.

   Select **Multi-protocol Share** to create a multi-protocol share (NFSv4/SMB). Set this if the path is shared through NFS, FTP, or used by containers or apps.
   Note: This setting can reduce SMB share performance as it turns off some SMB features for safer interoperability with external processes.
   See [Setting Up SMB Multichannel](# <!-- TODO-REF: smb-multichannel.md -->) for more information on creating multi-protocol shares.

   Select **Time Machine Share** to create a Time Machine share. The SMB share is presented to Mac OS clients as a Time Machine target.
   See [Adding a Basic Time Machine SMB Share](# <!-- TODO-REF: set-up-basic-time-machine-smb-share.md -->) for more information on creating and using Time Machine shares.

   Select **Final Cut Pro Storage Share** (available in TrueNAS 25.10.1 and later) to create a share optimized for Final Cut Pro workflows.
   The SMB share is configured with Apple-style character encoding and requires Apple SMB2/3 protocol extensions for compatibility with Final Cut Pro.
   See [Setting Up Final Cut Pro SMB Shares](# <!-- TODO-REF: fcp-share.md -->) for more information on creating shares for Final Cut Pro workflows.

   Select **External Share** to create an external share. See [Setting Up External SMB Shares](/shares/smb/external-smb-share) for more information. Enter the full domain name or IP address and the share name as *192.168.0.200\SHARE* in **Remote Path**.

   Select **Time Locked Share** to create a share that makes files read-only after the grace period you specify expires.
   This setting does not work if the path is accessed locally or if another SMB share with the **Time Locked Share** purpose uses the same path.
   Warning: This setting might not meet regulatory requirements for write-once storage.

4. (Optional) Enter a short description or explanation of the share purpose or use in **Description**.
   This shows on both the SMB widget and **Share > SMB** screen to help explain how the share is used.
   For example, if for an external share, enter *external share* in the field.
   The description entered shows in the **SMB** table on the **SMB** screen and the **Windows (SMB) Share** widget.

5. Select **Enabled** to allow sharing of this path when the SMB service is activated.
   Leave the checkbox cleared to disable the share without deleting the configuration.

6. (Optional) Click **Advanced Options** to show additional configuration settings.
   Click to configure other advanced settings such as access, audit logging, or settings specific to the type of share selected in **Purpose**.

7. Click **Save** to create the share and add it to the **Shares > Windows (SMB) Shares** list.

Start or restart the SMB service when prompted.
