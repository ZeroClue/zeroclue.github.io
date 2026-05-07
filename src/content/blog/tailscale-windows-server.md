---
title: "Tailscale on Windows Server"
date: 2026-05-07
description: "Putting your Windows servers on a private mesh network. No public IPs, no firewall rules, no port forwarding. Install Tailscale, authenticate, and your machines see each other as if they were on the same LAN."
draft: false
---

This is part of a series on [managing Windows Server Core without RDP](/blog/managing-windows-server-core-without-rdp).

---

SSH gives you a shell. But first you need to reach the machine. On a local network that's trivial — the IP is right there. When your servers are spread across different hosts, different networks, different cloud providers, reaching them becomes the problem.

You can open ports. Forward 22 through your firewall, point it at the server, and hope nobody probes it. You can set up a VPN. You can use a jump box. All of these work, and all of them involve configuration that you then have to maintain.

Tailscale takes a different approach. It puts every machine you authorize onto the same virtual network — a tailnet. Once a machine is on the tailnet, it can reach every other machine on it. No public IPs. No firewall rules. No port forwarding. Your servers see each other by name, as if they were plugged into the same switch.

## Install on Server Core

There's no GUI installer needed. Download and run:

```powershell
Invoke-WebRequest -Uri "https://pkgs.tailscale.com/stable/tailscale-setup-latest.exe" `
    -OutFile "$env:TEMP\tailscale-setup-latest.exe"
Start-Process -FilePath "$env:TEMP\tailscale-setup-latest.exe" -ArgumentList "/S" -Wait
```

The `/S` flag runs it silently. On Server Core there's nothing to show anyway.

## Authenticate with an auth key

On a headless server there's no browser to pop up and log into your Tailscale account. Instead, generate an auth key from the [Tailscale admin console](https://login.tailscale.com/admin/settings/keys) and use it to authenticate from the command line:

```powershell
& "C:\Program Files\Tailscale\tailscale.exe" up --authkey "tskey-auth-..." --unattended
```

The `--unattended` flag tells Tailscale not to try to open a browser. The auth key handles identification. The machine joins your tailnet and is immediately reachable from every other node on it.

## Connect from your client

Install Tailscale on your local machine the same way — download, install, authenticate. Once both machines are on the tailnet, you can SSH using the Tailscale hostname instead of an IP:

```bash
ssh deploy@servername
```

No IP addresses to remember, no DNS to configure. Tailscale assigns each machine a name based on its hostname and resolves it across the tailnet automatically.

## Verify the connection

From any machine on the tailnet:

```bash
tailscale status
```

You'll see every node, its Tailscale IP, and its hostname. From there, SSH works the same as it would on a local network.

## Why not just open port 22

You could. SSH is encrypted, the auth is solid, and plenty of people run exposed SSH servers. But every open port is an attack surface. Even if SSH itself is secure, you're betting that there's no bug in the implementation, no misconfiguration in the firewall, no credential that gets leaked.

Tailscale removes that bet. Your SSH server is only reachable from machines on your tailnet — machines you've explicitly authorized. Nothing on the public internet can see port 22 because it's not exposed. The network layer handles access control before SSH even sees the connection.

It also means no NAT traversal, no dynamic DNS, no wrestling with cloud provider security groups. The machine is just on your network.

## What's next

With Tailscale handling the network and [SSH handling the shell](/blog/ssh-powershell-windows-server-core), you have everything you need to manage Windows Server Core without RDP. No desktop required.
