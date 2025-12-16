---
title: "[D-72] Locked up state by VS Code [Remote SSH]"  
categories: [Thesis]  
tags:
- [log, thesis]

toc: true
toc_sticky: true
date: 2025-12-15 21:00
---

# Remote-SSH Connection Instability Report

This issue describes a failure in establishing a stable SSH connection 
from VS Code using the Remote-SSH extension, 
leading to repeated disconnections and a stalled, "locked up" state.

This problem did not occur with standard SSH, suggesting the root cause is specific 
to how the VS Code extension manages the persistent connection and server-side setup.

The disruption is likely caused by lingering `vscode-server` processes on the remote host, 
environment misconfigurations, or network instability that prevents the dedicated communication channel from being maintained.

Resolving the issue requires manual intervention to clear server-side processes and 
check local SSH configuration files. 

A stable connection can be rebuilt by systematically troubleshooting the server environment 
and local configuration.

- *Problem*: VS Code Remote-SSH disconnects shortly after opening, 
resulting in a persistent stalled/locked state upon reconnection attempts.

- *Likely Causes*: Lingering `vscode-server` processes on the remote machine, 
exhausted server disk space, or a conflict in local SSH configuration (`known_hosts`).

- *Resolution*: Kill any remaining vscode-server processes on the remote host and 
check the VS Code Remote - SSH output log for specific error messages (e.g., shell startup errors).

- *Immediate Action*: Attempt direct SSH login and execute `pkill -9 -f vscode-server`.
