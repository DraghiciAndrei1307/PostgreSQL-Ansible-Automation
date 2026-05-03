# Connect to the Windows host via SSH key

To connect to the windows host via SSH, I followed this tutorial: 
https://www.hanselman.com/blog/how-to-ssh-into-a-windows-10-machine-from-linux-or-windows-or-anywhere

Basically, what you need to excute are the following commands?

- check what `OpenSSH` stuff like is enabled. You should check the `OpenSSH.Server~~~~0.0.1.0` with state `NotPresent` 

```commandline
Get-WindowsCapability -Online | ? Name -like 'OpenSSH*'
```

This means that we can SSH from the Windows, but we cannot SSH to it.

- install the OpenSSH server

```commandline
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

If this step takes too long, just `CTRL+C`, close the PowerShell window and try this step again using a new PowerShell 
window which you `Run as administrator`

After this step is complete, just use this to start the SSH server:

```commandline
Start-Service sshd
```

Then make sure it is running:

```commandline
Get-Service sshd
```

Now you can SSH to the Windows machine.

An important thing we need to do next is to configure the Firewall. 

Before we configure the firewall, we neet to set the PowerShell as the default option for the OpenSSH. 

To do that use: 

```powershell
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH"
  -Name DefaultShell
  -Value "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" 
  -PropertyType String 
  -Force
```

After, we can check the current firewall rules by using: 

```commandline
> Get-NetFirewallRule -Name *ssh*

```

This is what we get

```powershell

Name                          : OpenSSH-Server-In-TCP
DisplayName                   : OpenSSH SSH Server (sshd)
Description                   : Inbound rule for OpenSSH SSH Server (sshd)
DisplayGroup                  : OpenSSH Server
Group                         : OpenSSH Server
Enabled                       : True
Profile                       : Private
Platform                      : {}
Direction                     : Inbound
Action                        : Allow
EdgeTraversalPolicy           : Blocks
LooseSourceMapping            : False
LocalOnlyMapping              : False
Owner                         :
PrimaryStatus                 : OK
Status                        : The rule was parsed successfully from the store. (65536)
EnforcementStatus             : NotApplicable
PolicyStoreSource             : PersistentStore
PolicyStoreSourceType         : Local
RemoteDynamicKeywordAddresses : {}
PolicyAppId                   :
PackageFamilyName             :

```

When we installed the OpenSSH server on the Windows host, we enabled the connections on port 22. 

Before we start configure the SSH server so that we can use SSH keys to connect to the Windows host, we need to copy 
the public key from the Linux VM to the Windows host:

On Linux VM, copy the public key:
```commandline
cat ~/.ssh/id_rsa.pub
```

On Windows host, do these in powerShell as administrator: 

- create `C:\ProgramData\ssh\administrators_authorized_keys` and paste de public SSH key there.
- configure the access rights on the `C:\ProgramData\ssh\administrators_authorized_keys` file:

```commandline
icacls "C:\ProgramData\ssh\administrators_authorized_keys" /inheritance:r
icacls "C:\ProgramData\ssh\administrators_authorized_keys" /grant "NT AUTHORITY\SYSTEM:(F)"
icacls "C:\ProgramData\ssh\administrators_authorized_keys" /grant "BUILTIN\Administrators:(F)"
```

- edit the `C:\ProgramData\ssh\sshd_config` file so that it contains the following lines :

```commandline

PubkeyAuthentication yes

Match Group administrators
    AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```

- cleaning the file (in case the wrong encoding or invisible chars occur)

```commandline
$content = Get-Content C:\ProgramData\ssh\administrators_authorized_keys
[System.IO.File]::WriteAllLines("C:\ProgramData\ssh\administrators_authorized_keys", $content)

```

- restart the sshd service

```commandline
Restart-Service sshd
```