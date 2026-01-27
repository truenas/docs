To connect to an SMB share, start the SMB service.

After adding a new share, TrueNAS prompts you to start or restart the SMB service.

You can also start the service from the **Windows (SMB) Share** widget or on the **System > Services** screen from the **SMB** service row.

### Starting the Service Using the Windows SMB Share

From the **Sharing** screen, click on the **Windows (SMB) Shares** <span class="material-icons">more_vert</span> to display the service options, which are **Turn Off Service** if the service is running or **Turn On Service** if the service is not running.

{{< trueimage src="/images/shares/smb-share-options.png" alt="SMB Service Options" id="SMB Service Options" >}}

Each SMB share on the list also has a toggle to enable or disable the service for that share.

### Starting the Service Using System Settings

To make SMB share available on the network, go to **System > Services** and click the **SMB** <span class="iconify" data-icon="mdi:play-circle" title="Start Service">Start Service</span> button to start the service.
Toggle **Start Automatically** on if you want the service to activate when TrueNAS boots.
