# Pipes, Redirection and Bash

A practical reference for connecting commands, controlling output, using shell variables, and writing small Bash command sequences.

## Quick Reference

| Task | Syntax | Meaning |
|---|---|---|
| Overwrite output file | `command > file` | Redirect stdout; replace existing content |
| Append output | `command >> file` | Add stdout to the end |
| Save errors | `command 2> errors.txt` | Redirect stderr |
| Append errors | `command 2>> errors.txt` | Append stderr |
| Save stdout and stderr | `command > output.txt 2>&1` | Both streams go to the file |
| Discard errors | `command 2>/dev/null` | Hide stderr |
| Pipe output | `command1 \| command2` | Connect stdout to stdin |
| Run on success | `command1 && command2` | Run second command if first succeeds |
| Run on failure | `command1 \|\| command2` | Run second command if first fails |
| Show last exit status | `echo $?` | Status of most recent command |
| Store command output | `result=$(command)` | Command substitution |
| Export variable | `export NAME=value` | Make variable available to child processes |

## 1. Standard Streams

### 1.1 stdin, stdout, and stderr

| File descriptor | Name | Default destination |
|---|---|---|
| `0` | Standard input (`stdin`) | Keyboard or redirected input |
| `1` | Standard output (`stdout`) | Terminal |
| `2` | Standard error (`stderr`) | Terminal |

```bash
cat file.txt
```

The command reads the file and writes its output to stdout.

## 2. Output Redirection

### 2.1 Redirect stdout and stderr

```bash
# Overwrite stdout
command > output.txt

# Append stdout
command >> output.txt

# Write stderr to a file
command 2> errors.txt

# Append stderr
command 2>> errors.txt

# Save stdout and stderr together
command > output.txt 2>&1

# Discard stderr
command 2>/dev/null
```

### 2.2 Redirection Order Matters

```bash
command > output.txt 2>&1
command 2>&1 > output.txt
```

In the first command, stdout is redirected to the file and stderr is then sent to the same destination. In the second, stderr is first connected to the original stdout destination, then stdout is redirected to the file. The two streams can therefore go to different destinations.

### 2.3 Save Output and Errors Separately

```bash
command > output.txt 2> errors.txt
```

This is useful when you need successful output and diagnostic messages separately.

## 3. Input Redirection and Here-Documents

### 3.1 Read Input from a File

```bash
sort < names.txt
```

### 3.2 Supply Multiple Lines with a Here-Document

```bash
cat <<EOF
First line
Second line
EOF
```

A here-document supplies text until the terminating delimiter. A quoted delimiter prevents variable and command expansion:

```bash
cat <<'EOF'
This remains literal: $HOME
EOF
```

## 4. Pipes

### 4.1 Connect Commands

A pipe (`|`) connects the stdout of one command to the stdin of the next.

```bash
ps aux | grep ssh
cut -d: -f1 /etc/passwd
find /etc -type f 2>/dev/null | sort
```

The first example may match the `grep` process itself. A pipeline normally passes stdout, not stderr.

### 4.2 Pipe stdout and stderr in Bash

```bash
command |& less
```

In Bash, `|&` connects both stdout and stderr to the next command.

### 4.3 Useful Pipelines

```bash
# Count account entries
wc -l < /etc/passwd

# Count repeated lines
sort names.txt | uniq -c

# Search and show line numbers
grep -Rni "pattern" /path 2>/dev/null | less
```

## 5. Command Chaining

| Operator | Behaviour |
|---|---|
| `;` | Run the next command regardless of the previous command's success |
| `&&` | Run the next command only if the previous command succeeds |
| `||` | Run the next command only if the previous command fails |
| `\|` | Pipe stdout to another command |

```bash
mkdir test; cd test
mkdir test2 && cd test2
cd /missing-directory || echo "Directory change failed"
```

Use `&&` when the next operation depends on the previous one succeeding.

## 6. Exit Status

### 6.1 Check a Command's Status

By convention, exit status `0` indicates success; a non-zero status indicates failure.

```bash
ls /etc
echo $?

ls /missing-directory
echo $?
```

`$?` contains the status of the most recently executed command. If you run `echo $?` again, the most recent command is now `echo`.

### 6.2 Pipeline Status

By default, Bash normally reports the status of the last command in a pipeline:

```bash
false | true
echo $?
```

This returns the status of `true`, even though `false` failed.

To make a pipeline report failure when a component fails:

```bash
set -o pipefail
```

This changes how pipeline success is evaluated. Use it deliberately in scripts.

## 7. Variables and Environment

### 7.1 Create and Read Variables

```bash
name="tester"
echo "$name"
echo "$HOME"
echo "$PATH"
env
```

Do not put spaces around `=` when assigning a shell variable:

```bash
name="tester"    # Correct
name = "tester"  # Incorrect
```

### 7.2 Export Variables

```bash
name="tester"
export name

# Or assign and export in one step
export LAB_DIR="/tmp/lab"
```

A shell variable is not automatically inherited by child processes unless exported.

### 7.3 Quoting

```bash
file="my report.txt"
cat "$file"

echo '$HOME'   # Literal text
echo "$HOME"   # Expanded variable
```

Double quotes around variable expansions help preserve spaces and avoid unintended word splitting.

## 8. Command Substitution and Arithmetic

### 8.1 Command Substitution

```bash
current_user=$(whoami)
kernel=$(uname -r)
echo "$current_user"
echo "$kernel"
```

Command substitution inserts a command's output into another command.

### 8.2 Arithmetic Expansion

```bash
count=3
echo $((count + 1))
```

Arithmetic expansion uses `$((...))` in Bash and compatible shells.

## 9. Simple Bash Scripts

### 9.1 Minimal Script Structure

```bash
#!/usr/bin/env bash

echo "User: $(whoami)"
echo "Host: $(hostname)"
echo "Kernel: $(uname -r)"
```

Save the script, then run it with Bash:

```bash
bash info.sh
```

Or make it executable and run it directly:

```bash
chmod +x info.sh
./info.sh
```

### 9.2 Read a Value into a Variable

```bash
read -r -p "Enter a filename: " filename
printf 'Selected: %s\n' "$filename"
```

Use `printf` when you need predictable formatted output. Quote variable expansions unless you intentionally want shell word splitting or glob expansion.

## 10. Common Mistakes

- `>` overwrites; `>>` appends.
- `2>/dev/null` hides stderr, not stdout.
- Redirection order changes where streams go.
- A pipe normally passes stdout only.
- `&&` and `||` depend on exit status, not on whether output looks correct.
- `$?` changes after every command.
- `name=value` must not contain spaces around `=`.
- Quote variables and filenames containing spaces.
- A pipeline may hide an earlier command's failure unless `pipefail` is enabled.

## Related Notes

- [Kali and Shell Basics](01-Kali-and-Shell-Basics.md)
- [Filesystem, Search and Text Processing](02-Filesystem-Search-and-Text-Processing.md)
- [Quick Reference](Quick-Reference.md)
