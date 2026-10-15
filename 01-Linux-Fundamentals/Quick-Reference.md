
# Linux Quick Reference

A compact command reference for Linux administration, OSCP preparation, and initial system inspection.

Use the topic files for explanations, examples, and common mistakes.

## 1. System and User Information

| Task | Command |
|---|---|
| Show OS information | `cat /etc/os-release` |
| Show kernel version | `uname -r` |
| Show system architecture | `uname -m` |
| Show hostname | `hostname` |
| Show current user | `whoami` |
| Show user and group IDs | `id` |
| Show current directory | `pwd` |
| Show command path | `command -v command` |

## 2. Files and Directories

| Task | Command |
|---|---|
| List files and permissions | `ls -la` |
| Display readable file sizes | `ls -lh` |
| Identify file type | `file filename` |
| Read a long file | `less filename` |
| Show first 20 lines | `head -n 20 filename` |
| Show last 20 lines | `tail -n 20 filename` |
| Follow a growing log | `tail -f filename` |
| Find a file by name | `find / -name "filename" 2>/dev/null` |
| Find files by extension | `find /path -type f -name "*.conf" 2>/dev/null` |

## 3. Text Processing

| Task | Command |
|---|---|
| Search recursively, ignoring case | `grep -Rni "pattern" /path 2>/dev/null` |
| Count lines, words, and bytes | `wc filename` |
| Count matching lines | `grep -c "pattern" filename` |
| Extract the first field | `cut -d: -f1 filename` |
| Sort lines | `sort filename` |
| Count repeated lines | `sort filename \| uniq -c` |
| Print selected columns | `awk '{print $1}' filename` |

## 4. Users and Permissions

| Task | Command |
|---|---|
| Show user and groups | `id` |
| List local accounts | `cat /etc/passwd` |
| Show current user's groups | `groups` |
| Check sudo permissions | `sudo -l` |
| Show file permissions | `ls -l filename` |
| Change file permissions | `chmod 640 filename` |
| Change file owner | `sudo chown user:group filename` |
| Find SUID files | `find / -type f -perm -4000 2>/dev/null` |
| Find SGID files | `find / -type f -perm -2000 2>/dev/null` |
| Find file capabilities | `getcap -r / 2>/dev/null` |

## 5. Processes and Services

| Task | Command |
|---|---|
| List processes | `ps aux` |
| Display process tree | `pstree -p` |
| Monitor processes | `top` |
| Show system services | `systemctl list-units --type=service` |
| Show service status | `systemctl status service-name` |
| Show listening sockets | `ss -lntup` |
| Show recent system logs | `journalctl -n 100 --no-pager` |
| Show disk usage | `df -h` |
| Show directory sizes | `du -sh /path` |
| Show memory usage | `free -h` |

## 6. Networking

| Task | Command |
|---|---|
| Show IP addresses | `ip addr` |
| Show routes | `ip route` |
| Show DNS configuration | `cat /etc/resolv.conf` |
| Test name resolution | `getent hosts example.com` |
| Test connectivity | `ping -c 4 IP_ADDRESS` |
| Show listening TCP and UDP sockets | `ss -lntup` |
| Show network interfaces | `ip link` |
| Test an HTTP endpoint | `curl -i http://TARGET` |

## 7. File Transfer

| Task | Command |
|---|---|
| Download a file | `wget http://HOST/file` |
| Download with curl | `curl -o file http://HOST/file` |
| Copy over SSH | `scp file user@HOST:/path/` |
| Copy from a remote host | `scp user@HOST:/path/file .` |
| Create a compressed archive | `tar -czf archive.tar.gz directory/` |
| List archive contents | `tar -tzf archive.tar.gz` |
| Extract an archive | `tar -xzf archive.tar.gz` |

## 8. Bash and Redirection

| Task | Syntax | Meaning |
|---|---|---|
| Redirect output | `command > file` | Overwrite a file |
| Append output | `command >> file` | Append to a file |
| Redirect errors | `command 2> errors.txt` | Save stderr |
| Save output and errors | `command > output.txt 2>&1` | Save both streams |
| Pipe output | `command1 \| command2` | Pass stdout to another command |
| Run on success | `command1 && command2` | Run the second command if the first succeeds |
| Run on failure | `command1 \|\| command2` | Run the second command if the first fails |
| Show exit status | `echo $?` | Display the previous command's status |

## Important Reminders

- Linux filenames are case-sensitive.
- `>` overwrites; `>>` appends.
- `2>` redirects standard error.
- `2>/dev/null` discards standard error, which can hide useful diagnostic information.
- `apt update` refreshes package indexes; `apt upgrade` upgrades installed packages.
- `sudo -l` checks the current user's permitted sudo commands.
- A suspicious permission or process is an observation, not proof of an exploitable vulnerability.
- Commands such as `find /` may return permission errors or take time. Use them selectively.