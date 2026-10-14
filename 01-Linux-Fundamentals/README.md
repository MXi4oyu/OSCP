
# Linux Fundamentals

Practical Linux notes for OSCP preparation, penetration testing, and technical assessments.

This module focuses on using the Linux command line, inspecting a target system, understanding permissions, and collecting information that may be relevant to further investigation.

## Learning Objectives

After completing this module, you should be able to:

- Navigate the Linux filesystem and manage files from the command line.
- Search files and process text efficiently.
- Use pipes, redirection, and basic Bash features.
- Inspect users, groups, permissions, processes, and services.
- Check network configuration and transfer files between systems.
- Perform structured Linux enumeration and record useful findings.

## Contents

| File | Topics |
|---|---|
| [Quick Reference](Quick-Reference.md) | Frequently used commands and options |
| [Kali and Shell Basics](01-Kali-and-Shell-Basics.md) | Environment, paths, files, and editors |
| [Filesystem, Search and Text Processing](02-Filesystem-Search-and-Text-Processing.md) | `find`, `grep`, `awk`, `sed`, and related tools |
| [Pipes, Redirection and Bash](03-Pipes-Redirection-and-Bash.md) | Streams, pipelines, variables, and command execution |
| [Users, Permissions and Privilege Checks](04-Users-Permissions-and-Privilege-Checks.md) | Accounts, permissions, sudo, SUID, and capabilities |
| [Processes, Services and System Inspection](05-Processes-Services-and-System-Inspection.md) | Processes, services, logs, storage, and system configuration |
| [Networking and File Transfer](06-Networking-and-File-Transfer.md) | IP configuration, connections, SSH, and file transfer |
| [Linux Enumeration Checklist](Linux-Enumeration-Checklist.md) | A structured inspection workflow |
| [Practice Labs](labs/linux-basic-practice.md) | Hands-on exercises and completion criteria |

## How to Use These Notes

1. Read the relevant topic file when learning a concept.
2. Use `Quick-Reference.md` when you need to recall a command quickly.
3. Follow `Linux-Enumeration-Checklist.md` when inspecting a Linux system.
4. Complete the exercises in `labs/linux-basic-practice.md`.
5. Record findings and explain why each command is useful rather than memorising commands without understanding their output.

## Scope

This module covers Linux fundamentals and initial system inspection.

Detailed Linux privilege escalation techniques, exploit development, and challenge-specific workflows should be documented in their respective modules.

## Recommended Practice

Use a Linux virtual machine that you control. Practise with ordinary user permissions first, then compare the results with a privileged account where appropriate.

Always verify the target, current user, working directory, and command effects before performing potentially destructive operations.