---
title: "Managing Windows Server Core without RDP"
date: 2026-05-07
description: "Windows Server Core has no desktop. RDP works, but it's the wrong tool for compiling code, running builds, and managing services remotely. Here's what I use instead."
draft: false
---

I'm building a managed file transfer product. Part of that product is a Windows CLI agent — written in Rust, compiled on Windows, deployed to Windows servers. The build machine is a Windows Server Core instance: no desktop, no Start menu, no taskbar. Just a command prompt and a login screen.

My day-to-day is a Linux machine with tmux — multiple windows, multiple panes, everything a terminal away. When I need to build the Windows binary, I SSH from tmux into the Windows server, run the build, grab the artifact, and carry on. No context switch. No remote desktop. Just another pane.

There's a better way.

**Tailscale** connects the machines. Every server goes on the same virtual network. No public IPs, no firewall rules, no port forwarding — your machines see each other as if they were on the same LAN.

**SSH** gives you a remote shell. OpenSSH ships with Windows Server. Enable it, configure it, and you have an encrypted terminal session from any machine on your tailnet. Connection drops? Reconnect. Your shell history, your working directory, your running processes are still there.

**PowerShell 7** gives you a modern shell on Server Core. The default that ships with Windows is PowerShell 5.1 — functional but outdated. PowerShell 7 is the current version, and once you set it as the default for SSH, every remote session starts in it automatically.

The rest of this series covers each one:

- [SSH and PowerShell 7 on Windows Server Core](/blog/ssh-powershell-windows-server-core) — installing OpenSSH, setting PowerShell 7 as the default shell, configuring key-based authentication, and the ACL gotcha that will silently break your setup if you don't know about it.

- [Tailscale on Windows Server](/blog/tailscale-windows-server) — installing Tailscale on a headless server, authenticating with an auth key, and connecting your machines into a private mesh network.

If you're managing Windows servers and still using RDP for everything, SSH is already there. It's been reliable for thirty years and turns a headless server into just another terminal.
