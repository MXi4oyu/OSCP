# Users, Permissions and Privilege Checks

A practical reference for understanding Linux accounts, file permissions, sudo rules, and permission-related findings during system inspection.

This file focuses on identifying and understanding configurations. Detailed exploitation techniques belong in a separate Linux Privilege Escalation module.

## Quick Reference

| Task | Command | Note |
|---|---|---|
| Show current user | `whoami` | Effective username |
| Show UID, GID, and groups | `id` | More detail than `whoami` |
| Show groups | `groups` | Current user's group membership |
| List local accounts | `cat /etc/passwd` | Account database |
| Inspect permissions | `ls -l file` | Mode and ownership |
| Inspect metadata | `stat file` | Detailed file metadata |
| Change permissions | `chmod 640 file` | Changes mode bits |
| Change owner | `sudo chown user:group file` | Requires appropriate privilege |
| Check sudo rules | `sudo -l` | Current user's permitted commands |
| Find SUID files | `find / -type f -perm -4000 2>/dev/null` | Review results; not proof of a vulnerability |
| Find SGID files | `find / -type f -perm -2000 2>/dev/null` | Review results |
| Find capabilities | `getcap -r / 2>/dev/null` | `getcap` may not be installed |

## 1. Users and Groups

### 1.1 Essential Commands and Files

```bash
whoami
id
groups
cat /etc/passwd
cat /etc/group
```

| Command or file | Purpose |
|---|---|
| `whoami` | Display the effective username |
| `id` | Display UID, GID, and supplementary groups |
| `groups` | Display group membership |
| `/etc/passwd` | Account information |
| `/etc/group` | Group information |
| `/etc/shadow` | Password hash and password-aging information; normally restricted |

### 1.2 Understand `/etc/passwd`

A typical entry has seven colon-separated fields:

```text
username:password_placeholder:UID:GID:GECOS:home_directory:login_shell
```

Example:

```text
alice:x:1000:1000:Alice:/home/alice:/bin/bash
```

The `x` commonly indicates that password information is stored elsewhere, typically in `/etc/shadow`. UID ranges and account purposes vary by system; do not classify accounts using UID alone.

## 2. Linux File Permissions

### 2.1 Read Symbolic Permissions

Example:

```text
-rwxr-xr--
```

| Part | Meaning |
|---|---|
| First character `-` | Regular file; `d` indicates a directory |
| `rwx` | Owner can read, write, and execute |
| `r-x` | Group can read and execute |
| `r--` | Others can read |

| Symbol | Meaning |
|---|---|
| `r` | Read |
| `w` | Write |
| `x` | Execute for files; search/traverse for directories |
| `-` | Permission not granted |

For directories, `x` allows traversal when the path is otherwise accessible. Read permission allows listing entries; write permission may allow creating or removing entries, subject to other restrictions.

### 2.2 Numeric Permissions

| Permission | Value |
|---|---:|
| Read (`r`) | 4 |
| Write (`w`) | 2 |
| Execute (`x`) | 1 |

| Mode | Meaning |
|---|---|
| `600` | Owner read/write |
| `640` | Owner read/write; group read |
| `644` | Owner read/write; group and others read |
| `700` | Owner full permissions |
| `750` | Owner full; group read/execute |
| `755` | Owner full; group and others read/execute |
| `777` | Read/write/execute for everyone |

### 2.3 Change Permissions

```bash
chmod 640 file.txt
chmod 750 script.sh
chmod u+x script.sh
chmod g-w file.txt
```

Use the narrowest permissions appropriate to the file's purpose.

## 3. Ownership

### 3.1 Inspect Ownership and Metadata

```bash
ls -l file.txt
stat file.txt
```

### 3.2 Change Ownership When Authorised

```bash
sudo chown alice file.txt
sudo chown alice:developers file.txt
```

`chown` changes the owner and, when specified, the group. Consider ownership, mode bits, and parent-directory permissions together. A file being writable by someone does not necessarily mean the current user can replace or delete it.

## 4. Sudo

### 4.1 Inspect Sudo Rules

```bash
sudo -l
```

Where authorised and permitted:

```bash
sudo -l -U username
```

The second command attempts to list another user's sudo privileges and generally requires sufficient permission.

### 4.2 What to Record

When reviewing `sudo -l`, record:

- The permitted command path.
- The run-as identity.
- Whether arguments are restricted.
- Whether environment variables are preserved.
- Whether the rule includes `NOPASSWD`.
- Whether a wildcard or broad command pattern is used.

A sudo rule is not automatically a vulnerability. Risk depends on the command, allowed arguments, configuration, and what the command can access or modify.

### 4.3 User Switching

```bash
sudo -i
sudo -s
su - username
```

These commands have different environment and authentication behaviours. Use them only when authorised and when you understand which identity and environment will be used.

## 5. SUID, SGID, and Capabilities

### 5.1 SUID and SGID

- **SUID:** For executable files, the process commonly runs with the file owner's effective UID.
- **SGID:** For executable files, the process commonly runs with the file's group as its effective GID.
- **SGID on directories:** Newly created entries commonly inherit the directory's group.

```bash
find / -type f -perm -4000 2>/dev/null
find / -type f -perm -2000 2>/dev/null
```

Inspect a result:

```bash
ls -l /path/to/file
file /path/to/file
```

### 5.2 File Capabilities

```bash
getcap -r / 2>/dev/null
```

Capabilities can grant specific privileges without granting full root privileges. Availability and output depend on the system and installed tools.

### 5.3 Evaluate Findings

For each unusual executable, permission bit, or capability, record:

- Path, owner, group, and mode.
- Purpose and version where known.
- Whether the current user can modify the executable, its parent directory, or its configuration.
- Evidence that demonstrates a real security impact.

A SUID/SGID bit or capability alone does not prove that the file is exploitable.

## 6. Sticky Bit

The sticky bit is commonly used on shared writable directories such as `/tmp`.

```bash
ls -ld /tmp
```

On a sticky directory, users generally cannot remove or rename entries owned by other users merely because the directory is writable. Exact behaviour is subject to filesystem and privilege rules.

## 7. Scheduled Tasks and Writable Scripts

### 7.1 Cron Locations

```bash
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /etc/cron.hourly/
ls -la /etc/cron.daily/
crontab -l
```

A cron entry commonly contains a schedule followed by a command. In `/etc/crontab`, an execution username is also specified.

Example:

```text
*/5 * * * * root /usr/local/bin/example.sh
```

This schedules the command every five minutes under the `root` account.

### 7.2 What to Inspect

Check the task's execution user, frequency, script path, owner, permissions, parent-directory permissions, `PATH`, and environment variables. Focus on whether a lower-privileged user can influence a task running with greater privileges.

## 8. Common Mistakes

- `whoami` does not show all group information; use `id`.
- `x` on a directory means traversal/search, not that the directory is an executable program.
- `chmod 777` is usually broader than necessary.
- File permissions and parent-directory permissions both matter.
- `sudo -l` lists permitted commands; it does not prove that a privilege escalation will work.
- SUID, SGID, and capabilities are investigation leads, not vulnerability conclusions.
- Locking password authentication does not necessarily disable SSH key authentication or existing sessions.

## Related Notes

- [Kali and Shell Basics](01-Kali-and-Shell-Basics.md)
- [Filesystem, Search and Text Processing](02-Filesystem-Search-and-Text-Processing.md)
- [Processes, Services and System Inspection](05-Processes-Services-and-System-Inspection.md)
- [Linux Enumeration Checklist](Linux-Enumeration-Checklist.md)
