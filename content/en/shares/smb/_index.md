---
title: "Windows Shares (SMB)"
description: "Provides information on SMB shares and instructions on creating a basic share and setting up various specific configurations of SMB shares."
geekdocCollapseSection: true
weight: 10
no_list: true
related: false
tags:
- smb
- afp
- shares
---

SMB (also known as CIFS) is the native file-sharing system in Windows.
SMB shares can connect to most operating systems, including Windows, Mac OS, and Linux.
TrueNAS can use SMB to share files among single or multiple users or devices.

SMB supports a wide range of permissions, security settings, and advanced permissions (ACLs) on Windows and other systems, as well as Windows Alternate Streams and Extended Metadata.
SMB is suitable for managing and administering large or small pools of data.

{{< include file="/static/includes/blocks/shares/smb.md" >}}
