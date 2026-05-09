---
title: "Migrating Claude Code from Windows to Linux"
date: 2026-05-09
description: "Moving a Claude Code instance — config, plugins, session history, Playwright profiles — from PowerShell on Windows to bash on Linux. What broke, what transferred, and what I'd do differently."
draft: false
---

I've been running Claude Code on Windows for about a month. PowerShell, `.claude-zcd/` config directory, the works. It worked. But "works" and "right" aren't the same thing, and last week I moved everything to a Linux VM.

Here's what happened.

## What needed to move

The config directory. That's where Claude Code keeps everything — settings, plugins, commands, session history, project memory. On Windows it was `~/.claude-zcd/`. On Linux it's `~/.claude-rook/` (different machine, different name). Copying the directory got most of it, but not all of it.

The vault. An Obsidian vault on a shared drive, accessible from both machines. No migration needed — the files are the same. But every path reference inside the vault was wrong. `N:\Kern\marketing\...` needs to be `~/projects/Kern/marketing/...`. PowerShell code blocks need to become bash. `python` becomes `python3`.

The scripts. Python utilities for image generation, PDF creation, file processing. All worked on Linux after changing `python` to `python3` and fixing one Windows-specific path in a config.

## What broke

**Playwright persistent profiles.** I had a Google-authenticated browser profile for automated tasks — LinkedIn posting, screenshot generation, that kind of thing. The profile copied over, but Chrome encrypts cookies with a per-machine key. Windows cookies don't decrypt on Linux. The profile looked intact but every session was unauthenticated. Had to create a fresh profile and re-login.

**Plugin paths.** The `installed_plugins.json` file had 22 Windows paths like `C:\Users\armin\.claude-zcd\plugins\cache\...`. All needed rewriting to `/home/arminm/.claude-rook/plugins/cache/...`. Without this fix, plugins would fail silently or reference non-existent directories.

**PowerShell-specific patterns.** Memory files had entries like "Windows Python: use `python`, not `python3`" and "PowerShell reserved variables: `$host` is read-only." Four entries needed removing. Two needed inverting for Linux.

**Permission accumulations.** The project `.claude/settings.local.json` had 13 stale Windows permissions — `pwsh` commands, `C:\Users\armin\` paths, `Get-Content *` (a PowerShell cmdlet), and two giant one-off Python image generation commands with Windows paths. All useless on Linux.

**Statusline script.** I had a PowerShell script that renders a status bar in the Claude Code terminal — git branch, context usage, rate limits. Needed a complete rewrite to bash. Used `jq` instead of Python for JSON parsing, unicode progress bars instead of ASCII, and a cleaner structure.

## What was easier than expected

**Scripts.** `python3 scripts/generate_carousels.py --list` just worked. No path changes, no import fixes, no encoding issues. The scripts were already platform-agnostic; the only difference was the Python binary name.

**Git and GitHub.** Global git config was already set to the right account. `gh auth` carried over via the config file. No issues.

**SSH deployments.** Tailscale connects to the VMs the same way from both platforms. `scp` commands are identical. No Windows-to-Linux translation needed.

## The noVNC detour

The Playwright profile problem led somewhere unexpected. No display on a headless Linux VM means no browser window for logging into Google. So I set up Xvfb + x11vnc + noVNC — a virtual display accessible from a web browser. The whole thing: install three packages, start a virtual X server, run a VNC server on it, expose it through websockify, and connect from any browser on the network.

Now I can open a browser on the VM from my Windows machine, log into whatever service needs authenticating, and close it. The Playwright persistent profile saves the session. It took longer to get working than I expected (stealth mode, DISPLAY variable scoping, profile encryption issues) but the result is a reusable tool I can direct whenever a login is needed.

## What I'd do differently

Copy the config directory first, then let Claude Code fix its own migration. Most of the work was finding Windows references and replacing them — the kind of pattern matching an AI agent does well. I spent too long manually checking each file before realising I should have just asked myself to sweep everything.

I'd also keep the config directory name consistent from the start. Using `claude-zcd` on Windows and `claude-rook` on Linux meant I had two separate plugin caches, two separate command directories, and confusion about which one was active. One name, one directory, one source of truth.

## How it feels

Better. Bash is faster for the kind of work I do — running scripts, managing files, deploying code. No more `python` vs `python3` debates. No more `Invoke-WebRequest` when `curl` would do. No more Windows path escaping in JSON strings. The tools are sharper and the friction is lower.

The migration itself took about two hours. Most of that was finding and fixing Windows-specific references across config files. If I'd planned it better, it could have been one.
