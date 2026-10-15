
# Linux Enumeration Checklist

A structured workflow for inspecting a Linux system during an authorised assessment.

The objective is to collect evidence, identify unusual configurations, and determine which findings require further investigation. Not every finding represents a vulnerability.

## 1. Establish the Current Context

- [ ] Confirm the target system.
- [ ] Identify the current user.
- [ ] Identify the user's groups and privileges.
- [ ] Record the current working directory.
- [ ] Record the hostname, distribution, kernel, and architecture.

```bash
whoami
id
pwd
hostname
cat /etc/os-release
uname -a
```

**Record:**
- Current user and group membership.
- Distribution and kernel version.
- Working directory and hostname.

## 2. Inspect the Filesystem

- [ ] List files, including hidden files.
- [ ] Inspect the current user's home directory.
- [ ] Identify accessible configuration files.
- [ ] Search relevant directories for interesting files.
- [ ] Check file ownership and permissions.

```bash
ls -la
ls -la ~
find /home -maxdepth 3 -type f 2>/dev/null
find /etc -maxdepth 2 -type f 2>/dev/null
```

Search results should be narrowed according to the assessment context. Avoid assuming that a file is sensitive or exploitable based only on its name.

**Record:**
- Unusual files and directories.
- Relevant configuration paths.
- Files with unexpected ownership or permissions.

## 3. Inspect Users and Privileges

- [ ] Identify local accounts.
- [ ] Inspect group membership.
- [ ] Check sudo permissions.
- [ ] Review relevant authentication and account configuration.
- [ ] Identify files or directories writable by the current user.

```bash
cat /etc/passwd
groups
sudo -l
find / -type f -writable 2>/dev/null
```

The writable-file search may be slow and produce many results. Prioritise relevant application directories, scripts, service configurations, and files used by privileged processes.

Do not attempt to read protected password data or access another user's private files unless the assessment explicitly authorises it.

**Record:**
- Accounts and groups of interest.
- Permitted sudo commands.
- Unexpected writable files or directories.
- Evidence supporting any privilege-related hypothesis.

## 4. Inspect Privileged File Mechanisms

- [ ] Identify SUID executables.
- [ ] Identify SGID executables.
- [ ] Check Linux file capabilities.
- [ ] Review relevant scheduled tasks and scripts.
- [ ] Determine whether any unusual permissions have a legitimate explanation.

```bash
find / -type f -perm -4000 2>/dev/null
find / -type f -perm -2000 2>/dev/null
getcap -r / 2>/dev/null
cat /etc/crontab
ls -la /etc/cron.d /etc/cron.daily 2>/dev/null
```

A SUID bit, capability, or scheduled task does not automatically indicate a vulnerability. Investigate the executable, its configuration, its ownership, and whether the current user can influence its behaviour.

**Record:**
- Executable path and owner.
- Relevant permission bits or capabilities.
- Associated scripts and configuration files.
- Whether any required conditions for abuse are actually present.

## 5. Inspect Processes and Services

- [ ] List running processes.
- [ ] Identify processes running under privileged accounts.
- [ ] Inspect service definitions and status.
- [ ] Identify listening TCP and UDP sockets.
- [ ] Review relevant scheduled jobs.
- [ ] Check whether scripts or configuration files used by services are writable.

```bash
ps aux
pstree -p
systemctl --type=service --state=running
ss -lntup
systemctl list-timers --all
```

Where permissions permit, inspect relevant service definitions and process command lines.

**Record:**
- Unusual processes and their execution context.
- Services running with elevated privileges.
- Listening ports and associated services.
- Potentially writable scripts or configurations used by those services.

## 6. Inspect System Resources and Logs

- [ ] Check filesystem capacity.
- [ ] Check memory usage.
- [ ] Inspect relevant application and system logs.
- [ ] Identify installed software relevant to the assessment.

```bash
df -h
free -h
journalctl -n 100 --no-pager
dpkg -l
```

On non-Debian systems, package-management commands will differ. Some logs may require elevated privileges.

**Record:**
- Relevant software and versions.
- Errors or unusual service activity.
- Any environmental constraints that affect further testing.

## 7. Inspect Network Configuration

- [ ] Identify network interfaces and IP addresses.
- [ ] Inspect the routing table.
- [ ] Review DNS configuration.
- [ ] Identify listening sockets.
- [ ] Check relevant local connectivity.

```bash
ip addr
ip route
cat /etc/resolv.conf
ss -lntup
getent hosts example.com
```

A failed connectivity test does not prove that a host or service is unavailable. Firewalls, DNS configuration, routing, and local restrictions may affect results.

**Record:**
- IP addresses and interfaces.
- Routes and gateways.
- DNS servers and resolution results.
- Listening ports and service associations.

## 8. Identify Follow-up Questions

For every potentially interesting observation, ask:

1. What exactly did the command reveal?
2. Which user can access or modify the resource?
3. Does the resource run with elevated privileges?
4. Can the current user influence its inputs, files, or execution?
5. What additional evidence is required to confirm the finding?

Do not jump directly from an unusual configuration to an exploitation conclusion.

## 9. Record Findings

Use a table to preserve the evidence and the next action.

| Area | Observation | Evidence | Next step |
|---|---|---|---|
| Users | Unusual group membership | Output of `id` | Inspect the group's permissions |
| Sudo | Permitted command found | Output of `sudo -l` | Review the command and its permitted arguments |
| Files | Unexpected writable script | File permissions and ownership | Identify whether a privileged process uses it |
| Processes | Unusual privileged service | Process and service details | Inspect its configuration |
| Network | Local listening port | Output of `ss -lntup` | Identify the service and its purpose |

Replace the example observations with actual findings. Never record an unverified assumption as a confirmed vulnerability.

## Completion Criteria

- [ ] System identity and user context are documented.
- [ ] Relevant filesystem and permission findings are recorded.
- [ ] Processes, services, and network configuration are inspected.
- [ ] Interesting observations have supporting evidence.
- [ ] Each finding has a clear next investigative step.
- [ ] No exploitation conclusion is based solely on a command's output without further validation.

## Related Notes

- [Quick Reference](Quick-Reference.md)
- [Filesystem, Search and Text Processing](02-Filesystem-Search-and-Text-Processing.md)
- [Users, Permissions and Privilege Checks](04-Users-Permissions-and-Privilege-Checks.md)
- [Processes, Services and System Inspection](05-Processes-Services-and-System-Inspection.md)
- [Networking and File Transfer](06-Networking-and-File-Transfer.md)
