# Kali and Shell Basics

A practical reference for Linux commands used during OSCP preparation and CTF practice. Examples generally work on Kali Linux and Ubuntu unless noted.

## Quick Reference

| Task | Command | Note |
|---|---|---|
| Identify distribution | `cat /etc/os-release` | Distribution and release |
| Check kernel | `uname -r` | Running kernel version |
| Identify current user | `whoami` / `id` | Username, UID, groups |
| Check command availability | `command -v nmap` | Checks the current `PATH` |
| Read command help | `man find` | Search with `/keyword`; quit with `q` |
| Search manual pages | `man -k passwd` | Search by keyword |
| List files and permissions | `ls -la` | Includes hidden files |
| Identify file type | `file filename` | Does not rely on the filename extension |
| Search for SUID files | `find / -perm -4000 -type f 2>/dev/null` | Permission-based enumeration |
| Search text | `grep -in 'pattern' file` | Case-insensitive; show line numbers |
| Show listening TCP/UDP sockets | `ss -lntup` | Some process details may require elevated privileges |
| Show network interfaces | `ip a` | Interface addresses |
| Show routes | `ip route` | Routing table |
| Print working directory | `pwd` | Current directory |
| Go to previous directory | `cd -` | Different from `cd ..` |
| Read a long file | `less file` | Search with `/pattern`; quit with `q` |
| Follow a log | `tail -f file` | Display appended lines |
| Save output | `command > output.txt` | Overwrites the file |
| Append output | `command >> output.txt` | Appends to the file |
| Suppress error messages | `command 2>/dev/null` | Hides standard error |
| Create a compressed archive | `tar -czf archive.tar.gz dir/` | gzip-compressed tar |
| List archive contents | `tar -tzf archive.tar.gz` | Inspect before extracting |
| Extract an archive | `tar -xzf archive.tar.gz` | Check destination first |

## 1. Basic Linux System Enumeration

Identify the system and current privileges before choosing the next enumeration steps.

### 1.1 Identify the Operating System and Kernel

```bash
cat /etc/os-release
uname -a
uname -r
hostname
```

| Command | Purpose |
|---|---|
| `cat /etc/os-release` | Distribution and release information |
| `uname -a` | Kernel, hostname, architecture, and other system information |
| `uname -r` | Running kernel version |
| `hostname` | Current hostname |

**Common mistake:** The distribution version and kernel version are different. Check both with `cat /etc/os-release` and `uname -r`. A version number alone does not prove that a vulnerability is present; patch level and configuration also matter.

![Ubuntu distribution and kernel version](images/linux-system-enumeration.png)

### 1.2 Identify the Current User and Privileges

```bash
whoami
id
```

| Command | Purpose |
|---|---|
| `whoami` | Current username |
| `id` | UID, primary GID, and supplementary groups |

Check group membership as well as the username. Group membership can affect access to files, devices, and administrative tools.

### 1.3 Check Network Interfaces and Routes

```bash
ip a
ip route
```

| Command | Purpose |
|---|---|
| `ip a` | Network interfaces and IP addresses |
| `ip route` | Routing table and default route |

### 1.4 Check Listening Services

```bash
ss -lntup
```

This lists listening TCP and UDP sockets. Process details may require elevated privileges.

## 2. Linux Manual Pages (`man`)

Use `man` to check unfamiliar command syntax and options. Manual pages or their index may be missing on minimal target systems.

### 2.1 Essential `man` Commands

```bash
# Read a command manual
man find

# Show help, if supported
find --help

# Search manual-page descriptions
man -k passwd

# Open a specific manual section
man 1 passwd
man 5 passwd
```

| Command | Purpose |
|---|---|
| `man command` | Open the default manual page |
| `command --help` | Display supported options, if available |
| `man -k keyword` | Search manual-page descriptions |
| `man section name` | Open a specific manual section |

### 2.2 Manual Sections to Recognise

Manual pages are grouped into sections. A name may appear in more than one section, because the sections document different kinds of entries. Use a section number when you need a specific entry.

| Section | Description | Example | OSCP / CTF relevance |
|---|---|---|---|
| `1` | User commands and executable programs | `man 1 passwd` | **High** — command syntax and options |
| `2` | System calls provided by the kernel | `man 2 open` | Medium — low-level program behaviour |
| `3` | Library functions, commonly C library functions | `man 3 printf` | Medium — program and library behaviour |
| `4` | Special files, usually found under `/dev` | `man 4 null` | Low to medium — device and special-file details |
| `5` | File formats and configuration files | `man 5 passwd` | **High** — configuration and file formats |
| `6` | Games and entertainment | `man 6 game-name` | Low — rarely relevant to core OSCP tasks |
| `7` | Miscellaneous topics, including conventions and protocols | `man 7 ip` | Medium — concepts, conventions, and protocols |
| `8` | System administration commands | `man 8 ip` | **High** — administrative and system-management tools |

Examples depend on which manual pages are installed. If an example is unavailable on your system, try another entry or search by keyword.

**Sections worth prioritising:**

- **Section 1:** Learn the syntax and options of commands such as `find`, `grep`, `ssh`, and `tar`.
- **Section 5:** Look up configuration and file formats, such as `passwd`, `shadow`, and `sudoers`.
- **Section 8:** Check system-administration tools where a separate section 8 page exists.
- **Sections 2 and 3:** Consult these when investigating system calls, library functions, or low-level program behaviour.
- **Sections 4, 6, and 7:** Recognise what they contain; consult them when a task makes them relevant.

```bash
# The passwd command
man 1 passwd

# The /etc/passwd file format
man 5 passwd

# Search for manual entries related to passwords
man -k passwd
```

**Important distinction:** The section number identifies the type of manual entry, not the command's importance or privilege level. Available entries and section coverage can differ between Linux distributions.

![Searching manual pages with man -k passwd](images/man-k-passwd-search.png)

### 2.3 Manual Page Navigation

| Key | Action |
|---|---|
| `/keyword` | Search forward for a word or pattern |
| `n` | Move to the next match |
| `N` | Move to the previous match |
| `Space` | Move forward one page |
| `b` | Move backward one page |
| `g` | Go to the beginning |
| `G` | Go to the end |
| `q` | Quit the manual |



### 2.4 Practical Example: Finding SUID Files

Use the manual to check the `find` permission syntax.

```bash
man find
```

Inside the manual, type `/-perm`, press `Enter`, then press `n` to find the next match. Press `q` to exit.

```bash
find / -perm -4000 -type f 2>/dev/null
```

| Part | Meaning |
|---|---|
| `/` | Start at the filesystem root |
| `-perm -4000` | Match files with the SUID permission bit set |
| `-type f` | Match regular files |
| `2>/dev/null` | Suppress error messages |

A SUID file is not automatically vulnerable. Review its owner, permissions, behaviour, and configuration before treating it as a possible privilege-escalation path.

**OSCP / CTF tip:** Use `man` to resolve uncertainty about an option. If the manual is unavailable, try `command --help` where supported.

## 3. Package Management

Kali Linux and Ubuntu both use APT and DPKG. In practice, first check whether a tool is already available. Install packages only when needed in your own lab environment.

### 3.1 Check Command and Package Availability

```bash
# Check whether a command is available in PATH
command -v nmap

# Check matching package status
dpkg -l | grep nmap
```

`command -v` checks whether a command can be found in the current shell's `PATH`. `dpkg -l` checks package status; a command may also have been installed outside APT.

### 3.2 Search and Inspect Packages

```bash
# Search package metadata
apt search nmap

# List files recorded for an installed package
dpkg -L nmap
```

| Command | Purpose |
|---|---|
| `apt search name` | Search available package metadata |
| `dpkg -L name` | List files recorded in the installed package database |

### 3.3 Install Packages

```bash
# Refresh package indexes; does not upgrade installed packages
sudo apt update

# Install a package
sudo apt install nmap
```

**OSCP / CTF tips:**

- Check whether a tool is already available before trying to install it.
- Kali and Ubuntu may provide different package versions or repository contents.
- A target may have no internet access or no permission to install packages.
- Avoid unnecessary package upgrades or other system changes on a target.

**Common mistake:** `apt update` refreshes package indexes; it does not upgrade installed packages.

## 4. Shell and Path Basics

Know where you are and how paths are interpreted.

### 4.1 Absolute and Relative Paths

| Symbol | Meaning |
|---|---|
| `/` | Filesystem root |
| `~` | Current user's home directory |
| `.` | Current directory |
| `..` | Parent directory |
| `-` in `cd -` | Previous working directory |

```bash
pwd
cd /etc
cd ..
cd ~
cd -
```

An absolute path starts from `/`, for example `/etc/passwd`. A relative path is interpreted from the current directory, for example `../file.txt`.

**Common mistake:** `cd ..` moves to the parent directory; `cd -` returns to the previous working directory.

### 4.2 Quoting and Special Characters

| Syntax | Meaning | Example |
|---|---|---|
| `'...'` | Preserve the literal meaning of characters inside single quotes | `grep 'user name' file.txt` |
| `"..."` | Preserve most characters literally while allowing variable and command expansion | `echo "$HOME"` |
| `\` | Escape the next character | `touch file\ name.txt` |
| `$VAR` | Expand a variable | `echo "$PATH"` |
| `$(command)` | Substitute command output | `echo "$(whoami)"` |

Quote paths or filenames containing spaces, for example `cat "my file.txt"`.

### 4.3 Useful Shell Commands

```bash
pwd                 # Show current directory
echo "$PATH"        # Show directories searched for commands
which python3       # Show a command path on many systems
command -v python3  # Preferred portable shell check
history             # Show command history in the current shell
```

A command not found in `PATH` may still exist elsewhere on the filesystem.

## 5. Listing and Viewing Files

Use these commands to inspect files, permissions, and logs during enumeration.

### 5.1 Listing Files

```bash
ls -la
ls -lh
file filename
```

| Command | Use |
|---|---|
| `ls -la` | List files, including hidden files, with permissions and ownership |
| `ls -lh` | Show file sizes in a readable format |
| `file filename` | Identify the file type |

### 5.2 Viewing Files and Logs

```bash
cat file.txt
less file.txt
head -n 20 file.txt
tail -n 20 file.txt
tail -f app.log
```

| Command | Use |
|---|---|
| `cat file` | Print a short file |
| `less file` | Read a long file; use `/pattern` to search and `q` to quit |
| `head -n 20 file` | Show the first 20 lines |
| `tail -n 20 file` | Show the last 20 lines |
| `tail -f file` | Follow newly appended log lines |

**Common mistake:** `cat` prints the whole file. For large logs or configuration files, use `less`, `head`, or `tail`.

### 5.3 Understanding Basic Permissions

Example permission string:

```text
-rwxr-xr--
```

| Part | Meaning |
|---|---|
| First character `-` | Regular file (`d` indicates a directory) |
| `rwx` | Owner can read, write, and execute |
| `r-x` | Group can read and execute |
| `r--` | Others can read |

Common permission characters:

- `r`: read
- `w`: write
- `x`: execute; for directories, it also relates to entering/traversing the directory

## 6. Basic File Operations

### 6.1 Create Files and Directories

```bash
# Create a file or update its timestamps
touch file.txt

# Create a directory
mkdir work

# Create nested directories
mkdir -p work/logs
```

`touch` creates an empty file if it does not exist, or updates timestamps if it does. It does not empty an existing file.

### 6.2 Copy, Move, and Rename

```bash
# Copy a file
cp source.txt destination.txt

# Copy a directory tree
cp -r source_dir/ destination_dir/

# Move or rename
mv old.txt new.txt
```

| Command | Purpose |
|---|---|
| `cp source dest` | Copy a file |
| `cp -r source_dir dest_dir` | Copy a directory tree |
| `mv source dest` | Move or rename a file or directory |

### 6.3 Delete Files and Directories

```bash
# Remove a file
rm file.txt

# Remove an empty directory
rmdir empty_dir

# Recursively remove a directory tree
rm -r directory/
```

**Safety reminder:** Check the path before using `rm`, especially with recursive deletion.

## 7. Archives and Compression

Archives combine files into one package; compression reduces data size. These are related but different operations. `tar` creates archives, while options such as `z` apply gzip compression.

### 7.1 Archive vs. Compression

- **Archive:** Combines files and directories into one file.
- **Compression:** Reduces the size of data.
- **`.tar`:** An archive that is not necessarily compressed.
- **`.tar.gz` / `.tgz`:** A tar archive compressed with gzip.
- **`.zip`:** A format that packages files and supports compression.

### 7.2 Create, List, and Extract `tar` Archives

```bash
# Create a gzip-compressed tar archive
tar -czf archive.tar.gz directory/

# List contents without extracting
tar -tzf archive.tar.gz

# Extract into the current directory
tar -xzf archive.tar.gz

# Extract into a specific destination directory
mkdir -p extracted/
tar -xzf archive.tar.gz -C extracted/
```

| Option | Meaning |
|---|---|
| `c` | Create an archive |
| `x` | Extract an archive |
| `t` | List archive contents |
| `f` | Specify the archive filename |
| `z` | Use gzip compression |
| `v` | Show filenames processed (verbose output) |
| `-C dir` | Change to `dir` before performing the operation |

Common format examples:

| Format | Typical command | Notes |
|---|---|---|
| `.tar` | `tar -cf archive.tar directory/` | Archive only; not compressed |
| `.tar.gz` / `.tgz` | `tar -czf archive.tar.gz directory/` | Tar archive compressed with gzip |
| `.tar.bz2` | `tar -cjf archive.tar.bz2 directory/` | Tar archive compressed with bzip2 |
| `.tar.xz` | `tar -cJf archive.tar.xz directory/` | Tar archive compressed with xz |

The order of short options is commonly flexible, but keep the archive filename immediately after `f` for clarity.

### 7.3 ZIP Archives

```bash
# List contents before extraction
unzip -l archive.zip

# Extract into the current directory
unzip archive.zip

# Extract into a chosen directory
unzip archive.zip -d extracted/

# Create a ZIP archive
zip -r archive.zip directory/
```

| Command | Purpose |
|---|---|
| `unzip -l archive.zip` | List ZIP contents |
| `unzip archive.zip` | Extract into the current directory |
| `unzip archive.zip -d directory/` | Extract into a chosen directory |
| `zip -r archive.zip directory/` | Create a ZIP archive including directory contents |

### 7.4 Other Compression Commands

```bash
gzip file.txt       # Compress to file.txt.gz; normally removes the original
gunzip file.txt.gz  # Decompress a .gz file
```

`gzip` compresses individual files; `tar` groups files and directories into an archive. They are often used together as `.tar.gz`.

### 7.5 Safe Extraction Workflow

1. **Identify the format:** use `file archive-name` if the extension is unclear.
2. **List contents first:** use `tar -tzf archive.tar.gz` or `unzip -l archive.zip`.
3. **Choose the destination:** extract into a new directory when handling unfamiliar archives.
4. **Check the result:** use `ls -la extracted/` or `find extracted/ -maxdepth 2 -type f`.

**Common mistakes:**

- `tar -czf` creates an archive; `tar -xzf` extracts one; `tar -tzf` lists contents.
- `.tar` does not necessarily mean compressed. The `z`, `j`, or `J` option selects gzip, bzip2, or xz compression respectively.
- Listing contents does not guarantee an archive is safe. Be careful with unfamiliar files and paths before extraction.

**OSCP / CTF tip:** Be comfortable creating, listing, and extracting `.tar.gz` and `.zip` files. These are common formats for transferring scripts, evidence, and challenge files.

## 8. Terminal Editors: Essential Commands

When editing files in a terminal, remember the basic save, search, and exit commands for `nano` and `vim`.

### 8.1 Nano

| Shortcut | Action |
|---|---|
| `Ctrl+O` | Save (write out); press `Enter` to confirm the filename |
| `Ctrl+X` | Exit |
| `Ctrl+W` | Search |
| `Ctrl+K` | Cut the current line |
| `Ctrl+U` | Paste the cut text |

### 8.2 Vim

Vim has different modes. Press `Esc` to return to Normal mode before entering commands.

| Action | Command |
|---|---|
| Enter Insert mode | `i` |
| Return to Normal mode | `Esc` |
| Save | `:w` |
| Save and quit | `:wq` |
| Quit without saving | `:q!` |
| Quit if there are no unsaved changes | `:q` |
| Delete current line | `dd` |
| Undo | `u` |
| Search forward | `/pattern`, then `Enter` |
| Repeat search | `n` |

**OSCP / CTF tips:**

- Use `nano` when you need to edit a configuration file or create a simple script quickly.
- In Vim, press `Esc` before entering commands such as `:wq` or `:q!`.
- Check whether an editor is available with `command -v nano vim vi`.
- Verify changes after editing, for example with `cat filename` or `less filename`.

## 9. Common Mistakes

- Linux filenames are case-sensitive.
- `>` overwrites a file; `>>` appends output.
- `2>/dev/null` hides error messages and can also hide useful diagnostic information.
- `apt update` refreshes package indexes; it does not upgrade installed packages.
- The distribution version is not the same as the running kernel version.
- `cd ..` and `cd -` do different things.
- `touch` does not empty an existing file.
- `rm -r` can delete entire directory trees.
- A SUID file is a lead to investigate, not proof of a vulnerability.
- A command being absent from `PATH` does not prove that its package is not installed.
- An archive extension does not guarantee the file's actual format; use `file` when unsure.
- Inspect unfamiliar archives before extracting them.

## Related Notes

- [Quick Reference](Quick-Reference.md)
- [Filesystem, Search and Text Processing](02-Filesystem-Search-and-Text-Processing.md)
- [Pipes, Redirection and Bash](03-Pipes-Redirection-and-Bash.md)
- [Linux Enumeration Checklist](Linux-Enumeration-Checklist.md)
