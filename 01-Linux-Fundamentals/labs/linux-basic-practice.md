
# Linux Basic Practice Labs

Hands-on exercises for building the Linux command-line skills required for OSCP preparation.

## Lab Environment

Use a Linux virtual machine that you control, such as Kali Linux or Ubuntu.

Before starting:

- [ ] Confirm that you are working in your own lab.
- [ ] Create a practice directory.
- [ ] Check your current user and working directory.
- [ ] Avoid modifying system configuration or deleting important files.

Initial setup:

```bash
mkdir -p ~/linux-practice/{files,logs,results}
cd ~/linux-practice
whoami
pwd
```

## Lab 1: System Identification

### Tasks

- [ ] Identify the Linux distribution and version.
- [ ] Identify the kernel version and architecture.
- [ ] Display the hostname and current user.
- [ ] Display your UID, GID, and supplementary groups.
- [ ] Save the results to a text file.

### Acceptance Criteria

You can explain the difference between distribution information, kernel information, and user identity.

## Lab 2: Files and Directories

### Tasks

- [ ] Create three nested directories.
- [ ] Create a text file in the deepest directory.
- [ ] Copy the file to another directory.
- [ ] Rename the copied file.
- [ ] List hidden files and inspect file permissions.
- [ ] Identify the file type.

Example:

```bash
mkdir -p files/project/logs
printf "Linux practice\n" > files/project/logs/example.txt
cp files/project/logs/example.txt results/copied.txt
mv results/copied.txt results/renamed.txt
ls -la results
file results/renamed.txt
```

### Acceptance Criteria

You can create, copy, move, rename, and inspect files without confusing absolute and relative paths.

## Lab 3: Search and Text Processing

Create a sample file:

```bash
cat > files/users.txt <<'EOF'
alice:1001:developers
bob:1002:admins
carol:1003:developers
dave:1004:admins
EOF
```

### Tasks

- [ ] Find the file by name.
- [ ] Search for all entries containing `admins`.
- [ ] Extract the usernames.
- [ ] Extract the numeric IDs.
- [ ] Count the number of lines.
- [ ] Count the number of entries in each group.

Example commands:

```bash
find files -name "users.txt"
grep -n "admins" files/users.txt
cut -d: -f1 files/users.txt
cut -d: -f2 files/users.txt
wc -l files/users.txt
cut -d: -f3 files/users.txt | sort | uniq -c
```

### Acceptance Criteria

You can select and transform text using `find`, `grep`, `cut`, `sort`, `uniq`, and `wc`.

## Lab 4: Pipes and Redirection

### Tasks

- [ ] Redirect command output to a file.
- [ ] Append another line without overwriting the file.
- [ ] Redirect standard error to a separate file.
- [ ] Pipe command output into `grep`.
- [ ] Use `&&` to connect two dependent commands.
- [ ] Check a command's exit status.

Example:

```bash
printf "alpha\nbeta\n" > results/output.txt
printf "gamma\n" >> results/output.txt
ls /path/that/does/not/exist 2> results/errors.txt
cat results/output.txt | grep "beta"
test -f results/output.txt && echo "File exists"
echo $?
```

### Acceptance Criteria

You can explain the difference between stdout, stderr, redirection, and pipes.

## Lab 5: Permissions

### Tasks

- [ ] Create a text file.
- [ ] Set its permissions to `600`.
- [ ] Change the permissions to `640`.
- [ ] Inspect the resulting permissions.
- [ ] Create a directory and explain how read, write, and execute permissions affect directory access.
- [ ] Inspect your current group membership.

Example:

```bash
touch files/permissions.txt
chmod 600 files/permissions.txt
ls -l files/permissions.txt
chmod 640 files/permissions.txt
ls -l files/permissions.txt
mkdir files/restricted
chmod 700 files/restricted
ls -ld files/restricted
id
```

### Acceptance Criteria

You can convert between numeric permissions and symbolic permissions, and explain how file permissions differ from directory permissions.

## Lab 6: Process and Service Inspection

### Tasks

- [ ] List running processes.
- [ ] Identify the PID and user for a selected process.
- [ ] Display the process tree.
- [ ] List listening TCP sockets.
- [ ] Check the status of a service available in your lab.
- [ ] Inspect recent system logs where permitted.

Example:

```bash
ps aux
pstree -p
ss -lnt
systemctl status ssh
journalctl -n 30 --no-pager
```

The SSH service may not be installed or running. If necessary, choose another service present in your lab.

### Acceptance Criteria

You can identify a process, inspect a service, and distinguish process information from listening-socket information.

## Lab 7: Network Inspection

### Tasks

- [ ] Identify the IP addresses assigned to the machine.
- [ ] Display the routing table.
- [ ] Display DNS resolver configuration.
- [ ] Resolve a hostname.
- [ ] Test a TCP endpoint in your lab.
- [ ] Explain why a failed ping does not prove a host is unavailable.

Example:

```bash
ip addr
ip route
cat /etc/resolv.conf
getent hosts example.com
```

Only test network endpoints you are authorised to access.

### Acceptance Criteria

You can explain the roles of IP addresses, routing, DNS, and transport-layer connectivity.

## Lab 8: Archive and Transfer

### Tasks

- [ ] Create a compressed tar archive of the practice directory.
- [ ] List the archive contents.
- [ ] Extract the archive into a separate directory.
- [ ] Calculate SHA-256 hashes for the original and extracted files.
- [ ] If a second authorised lab machine is available, transfer a test file using SSH-based tools.

Example:

```bash
tar -czf results/practice.tar.gz files/
tar -tzf results/practice.tar.gz
mkdir -p results/extracted
tar -xzf results/practice.tar.gz -C results/extracted
sha256sum files/users.txt results/extracted/users.txt
```

### Acceptance Criteria

You can archive, list, extract, transfer, and verify files.

## Final Self-Assessment

Mark a skill complete only when you can perform the task and explain the result without copying a solution.

- [ ] I can navigate the filesystem confidently.
- [ ] I can find files and search their contents.
- [ ] I can extract and transform text.
- [ ] I understand pipes and redirection.
- [ ] I can interpret Linux permissions.
- [ ] I can inspect processes and services.
- [ ] I can inspect IP addresses, routes, DNS, and listening ports.
- [ ] I can transfer files and verify their integrity.
- [ ] I can document findings with evidence.

## Review Notes

After completing the labs, record:

| Topic | What I learned | What I need to review |
|---|---|---|
| Filesystem | | |
| Text processing | | |
| Bash | | |
| Permissions | | |
| Processes and services | | |
| Networking | | |
| File transfer | | |

## Next Step

Once these exercises are complete, use the [Linux Enumeration Checklist](../Linux-Enumeration-Checklist.md) to perform a structured inspection of your lab system.

The goal is not just to remember commands. It is to understand what each command reveals, what its output does not prove, and which observation should be investigated next.
