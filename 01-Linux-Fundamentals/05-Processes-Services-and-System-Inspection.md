# Processes, Services and System Inspection

A practical reference for examining running processes, system services, scheduled tasks, logs, storage, and system configuration.

## Quick Reference

| Task | Command | Note |
|---|---|---|
| Show system identity | `hostname; uname -a` | Host and kernel |
| Show uptime and load | `uptime` | Current load averages |
| List processes | `ps aux` | Snapshot of running processes |
| Show process tree | `pstree -p` | Parent-child relationships |
| Monitor processes | `top` | Interactive view |
| Inspect a PID | `ps -p PID -o pid,ppid,user,group,stat,etime,args` | Replace `PID` |
| List running services | `systemctl list-units --type=service --state=running` | systemd systems |
| Inspect service | `systemctl status ssh` | Service name varies |
| List systemd timers | `systemctl list-timers --all` | Scheduled jobs |
| Show listening sockets | `ss -lntup` | Process details may require privilege |
| Read recent journal | `journalctl -n 100 --no-pager` | systemd journal |
| Check disk capacity | `df -hT` | Filesystem usage |
| Check memory | `free -h` | Memory and swap |
| List block devices | `lsblk` | Storage devices |

## 1. System Information

### 1.1 Identify the Host and Kernel

```bash
hostname
uname -a
uname -r
cat /etc/os-release
date
uptime
```

| Command | Purpose |
|---|---|
| `hostname` | System hostname |
| `uname -a` | Kernel and system information |
| `uname -r` | Running kernel version |
| `cat /etc/os-release` | Distribution and release |
| `date` | System date and time |
| `uptime` | Uptime and load averages |

Record distribution and kernel separately. A version number is a clue, not proof of a vulnerability.

### 1.2 Identify Logged-in Users

```bash
who
w
```

These show logged-in users and session activity. Availability and output vary by system.

## 2. Processes

### 2.1 List and Inspect Processes

```bash
ps aux
ps -ef
pstree -p
top
```

Useful process fields:

| Field | Meaning |
|---|---|
| PID | Process ID |
| PPID | Parent process ID |
| USER | User associated with the process |
| STAT | Process state |
| ELAPSED | Elapsed execution time |
| COMMAND | Command and arguments |

### 2.2 Inspect a Specific Process

```bash
ps -p PID -o pid,ppid,user,group,stat,etime,args
```

Replace `PID` with the process ID. When a process looks unusual, check its executable path, parent process, execution user, arguments, and associated service. A process name alone is not sufficient to identify its purpose.

## 3. Services and `systemctl`

### 3.1 List and Inspect Services

On systems using systemd:

```bash
systemctl list-units --type=service --state=running
systemctl status ssh
systemctl list-unit-files --type=service
```

Service names vary between distributions and installations. A service may exist without currently running.

### 3.2 Inspect a Service Definition

```bash
systemctl cat ssh
```

Where permitted, check:

- Which executable the service runs.
- Which user and group it uses.
- Whether it starts other processes.
- Which configuration files it reads.
- Whether relevant scripts or configuration files are writable.

## 4. Listening Ports and Local Services

### 4.1 Inspect Listening Sockets

```bash
ss -lnt
ss -lntu
ss -lntup
```

| Option | Meaning |
|---|---|
| `-l` | Listening sockets |
| `-n` | Numeric addresses and ports |
| `-t` | TCP sockets |
| `-u` | UDP sockets |
| `-p` | Process information, subject to permissions |

A listening socket does not, by itself, prove that the service is reachable from another host.

### 4.2 Link Ports to Services

Use `ss -lntup` and, where permissions allow, record the process name and PID. Then inspect the process and service definition. Do not infer an application's identity from its port number alone.

## 5. Scheduled Tasks

### 5.1 Cron

```bash
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /etc/cron.hourly/
ls -la /etc/cron.daily/
crontab -l
```

Example system crontab entry:

```text
*/5 * * * * root /usr/local/bin/example.sh
```

In `/etc/crontab`, the username field specifies the execution account. Per-user crontabs have a different field layout.

Inspect the task's script, owner, permissions, parent directory, environment, and any files it uses.

### 5.2 Systemd Timers

```bash
systemctl list-timers --all
```

For a timer of interest, inspect both the timer unit and its associated service unit where permissions permit. Not all scheduled work uses cron.

## 6. Logs

### 6.1 Common Log Sources

| Location or command | Purpose |
|---|---|
| `/var/log/` | Common system and application log directory |
| `/var/log/auth.log` | Authentication-related logs on many Debian-based systems |
| `/var/log/syslog` | General system messages on many Debian-based systems |
| `journalctl` | Read systemd journal entries |

### 6.2 Read Recent Logs

```bash
journalctl -n 100 --no-pager
journalctl -u ssh -n 100 --no-pager
journalctl --since "1 hour ago" --no-pager
```

Log locations and permissions vary. Some logs require elevated privileges.

When investigating an event, record the timestamp, service, event, and relevant surrounding context. A single error message may not explain the underlying cause.

## 7. Storage and Memory

### 7.1 Filesystem and Directory Usage

```bash
df -h
df -hT
du -sh /var/log/
findmnt
lsblk
mount
```

| Command | Purpose |
|---|---|
| `df -hT` | Filesystem capacity, usage, and type |
| `du -sh /path` | Total size of a directory |
| `findmnt` | Mounted filesystems and mount relationships |
| `lsblk` | Block devices |
| `mount` | Current mount information |

### 7.2 Memory

```bash
free -h
```

`free -h` displays memory and swap usage in readable units.

## 8. Installed Software

### 8.1 Debian-Based Systems

```bash
dpkg -l
dpkg -l | grep -i package
```

Package commands differ across distributions. Package presence does not necessarily prove that a service is running.

## 9. Common Mistakes

- Distribution version and kernel version are different.
- Process names alone do not establish a process's purpose.
- A listening port does not prove remote reachability.
- `systemctl` commands apply to systems using systemd.
- Cron and systemd timers are different scheduling mechanisms.
- Log locations and access permissions vary by distribution.
- Installed package status does not prove that a service is active.

## Related Notes

- [Kali and Shell Basics](01-Kali-and-Shell-Basics.md)
- [Users, Permissions and Privilege Checks](04-Users-Permissions-and-Privilege-Checks.md)
- [Networking and File Transfer](06-Networking-and-File-Transfer.md)
- [Linux Enumeration Checklist](Linux-Enumeration-Checklist.md)
