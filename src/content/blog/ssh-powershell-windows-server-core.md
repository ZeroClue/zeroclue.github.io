---
title: "SSH and PowerShell 7 on Windows Server Core"
date: 2026-05-07
description: "OpenSSH ships with Windows Server. Here's how to enable it, set PowerShell 7 as your default shell, configure key-based auth, and avoid the ACL gotcha that silently breaks everything."
draft: false
---

This is part of a series on [managing Windows Server Core without RDP](/blog/managing-windows-server-core-without-rdp).

---

Windows Server ships with OpenSSH. It's been the standard for remote shell access on every other platform for decades. On Windows it just happens to be turned off by default.

## Enable the SSH server

Log in to your Server Core machine and run:

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Set-Service -Name sshd -StartupType 'Automatic'
Start-Service sshd
```

That's it. SSH is running. You can connect from any machine with an SSH client. The default shell will be cmd.exe — the Command Prompt that ships with Windows. It works, but if you want PowerShell 7, keep reading.

## Install PowerShell 7

PowerShell 7 installs alongside the existing PowerShell 5.1 — it doesn't replace it. Both live on the machine. The trick is telling SSH to use the new one instead of cmd.exe.

```powershell
iex "& { $(irm https://aka.ms/install-powershell.ps1) } -UseMSI -Quiet"
```

Now set it as the default shell for SSH sessions:

```powershell
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell `
    -Value "C:\Program Files\PowerShell\7\pwsh.exe" `
    -PropertyType String -Force

Restart-Service sshd
```

Next time you SSH in, you'll land in PowerShell 7. Verify it:

```powershell
$PSVersionTable.PSVersion
```

## Create a dedicated user

Don't use the Administrator account for SSH. Create a user with a specific purpose:

```powershell
$password = Read-Host -AsSecureString
New-LocalUser -Name "deploy" -Password $password `
    -FullName "Deploy User" -Description "Remote build and deploy access"
Add-LocalGroupMember -Group "Administrators" -Member "deploy"
```

This user needs to be in the Administrators group if you want it to run builds and manage services. The principle is the same as on Linux — a dedicated account with the minimum privileges it needs.

## Key-based authentication

Passwords work, but keys are better. Generate an ED25519 key pair on your client machine:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_servername
```

Then copy the public key to the server. For an admin user, the key goes in `C:\ProgramData\ssh\administrators_authorized_keys`:

```bash
cat ~/.ssh/id_ed25519_servername.pub | ssh deploy@servername `
    '$Input | Add-Content -Path "$env:ProgramData\ssh\administrators_authorized_keys" -Encoding utf8'
```

## The ACL gotcha

This is the part that will waste your time if you don't know about it.

Windows OpenSSH checks the file permissions on `administrators_authorized_keys` before it reads the keys. If the permissions are too loose — if any user other than Administrators or SYSTEM can read or modify the file — SSH silently ignores all the keys in it. No error. No log message. Key auth just stops working, and you're left wondering why.

Fix it by locking down the ACL:

```powershell
$keyFile = "$env:ProgramData\ssh\administrators_authorized_keys"
$acl = Get-Acl $keyFile
$acl.SetAccessRuleProtection($true, $false)
$acl.Access | ForEach-Object { $acl.RemoveAccessRule($_) }
$acl.SetAccessRule([System.Security.AccessControl.FileSystemAccessRule]::new('Administrators', 'FullControl', 'Allow'))
$acl.SetAccessRule([System.Security.AccessControl.FileSystemAccessRule]::new('SYSTEM', 'FullControl', 'Allow'))
Set-Acl $keyFile $acl
```

Verify it took:

```powershell
icacls $keyFile
```

You should see exactly two entries:

```
NT AUTHORITY\SYSTEM:(F)
BUILTIN\Administrators:(F)
```

Nothing else. If either of these is missing, or if there's a third entry, key auth won't work.

## Test it

From your client machine:

```bash
ssh deploy@servername "echo hello"
```

If you see `hello` without a password prompt, it's working.

## Logging

If something isn't working and you need to debug, SSH logs go to the Windows Event Viewer under `OpenSSH/Operational`. For more detail, edit `C:\ProgramData\ssh\sshd_config`:

```
LogLevel DEBUG3
```

Restart the service, reproduce the issue, check the logs. Then change it back to `INFO` — debug logging fills up fast.

## What's next

SSH gets you a shell. Tailscale gets you a network. The next post covers [installing Tailscale on Windows Server](/blog/tailscale-windows-server) — putting your machines on a private mesh network without public IPs or firewall rules.
