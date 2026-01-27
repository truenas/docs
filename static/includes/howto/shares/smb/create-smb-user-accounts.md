{{< include file="/static/includes/local-smb-user.md" >}}

To add or edit users, go to **Credentials > Users**, then add or edit an existing user to create the SMB share user(s).
Click **Add** to create a new user or as many new user accounts as needed.
Joining TrueNAS to Active Directory creates the user accounts.

Enter the values in each required field, verify **SMB Access** is selected, then click **Save**.
For more information on the fields and adding users, see [Creating User Accounts](# <!-- TODO-REF: ManageUsers -->).

By default, all new users are members of a built-in group called **builtin_users**.
You can use a group to grant access to all users on the server or add more groups to fine-tune permissions for large numbers of users.

{{< expand "Why not just allow anonymous access to the share?" "v" >}}
Anonymous or guest access to the share is possible, but allowing guest access can create a security vulnerability, so it is not recommended for Enterprise customers or systems with more than one SMB share administrator account.
Using a guest account increases the likelihood of unauthorized users gaining access to your data in the SMB share.
Major SMB client vendors are deprecating guest users, partly because signing and encryption are impossible for guest sessions.
{{< /expand >}}

{{< expand "What about LDAP users?" "v" >}}
{{< hint type=important >}}
Support for LDAP **Samba Schema** is deprecated in TrueNAS 22.02 (Angelfish) and removed in 24.10 (Electric Eel).
Migrate legacy Samba domains to Active Directory before upgrading to 24.10 or later.
{{< /hint >}}
{{< /expand >}}
