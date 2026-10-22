# Linux Modules 1–4: Complete Command Cheat Sheet

This combines the Linux commands from your first four modules into one revision guide for your junior DevOps preparation. Each command includes its meaning and a practical example.

Use Ubuntu/WSL to practise. Commands that create, move, or delete files should be run inside your practice directory unless you intentionally need another location.

# Module 1 — Linux Fundamentals and Basic Commands

## 1. Identity and system information

| Command               | Meaning                                       |
| --------------------- | --------------------------------------------- |
| `whoami`              | Show the current username                     |
| `id`                  | Show user ID, group ID, and group memberships |
| `uname -s`            | Show the operating system kernel name         |
| `uname -r`            | Show the kernel version                       |
| `uname -a`            | Show detailed kernel/system information       |
| `cat /etc/os-release` | Show Linux distribution information           |
| `date`                | Show the current date and time                |
| `pwd`                 | Show the current working directory            |
| `clear`               | Clear the terminal display                    |
| `history`             | Show previously entered commands              |

Examples:

```
whoami
id
uname -a
cat /etc/os-release
date
```

## 2. Getting help

| Command          | Meaning                                    |
| ---------------- | ------------------------------------------ |
| `man ls`         | Open the manual for `ls`                   |
| `ls --help`      | Show command help and options              |
| `command --help` | Request help for a command, when supported |

Inside a manual page, press `q` to quit.

## 3. Users and permissions context

| Command        | Meaning                                               |
| -------------- | ----------------------------------------------------- |
| `sudo command` | Run a command with elevated privileges, if authorized |
| `su username`  | Switch to another user, if permitted                  |
| `su -`         | Switch to the root account, if permitted              |

Examples:

```
sudo apt update
su -
```

`sudo` and `su` may require permissions or authentication. Don't use elevated privileges unless needed.

## 4. Environment variables and command lookup

| Command             | Meaning                                                             |
| ------------------- | ------------------------------------------------------------------- |
| `echo "$HOME"`      | Display the home directory path                                     |
| `echo "$PATH"`      | Display directories searched for executable commands                |
| `env`               | Display exported environment variables                              |
| `printenv`          | Display environment variables                                       |
| `export NAME=value` | Set/export a variable for the current shell and its child processes |
| `command -v ls`     | Show how the shell resolves `ls`                                    |
| `which ls`          | Show the executable path, where `which` is installed                |

Example:

```
export APP_ENV=dev
echo "$APP_ENV"
```

Output:

```
dev
```

This variable normally lasts only for the current shell session unless you configure it to persist.

## 5. Processes and system services

| Command                    | Meaning                                     |
| -------------------------- | ------------------------------------------- |
| `ps`                       | Show processes associated with the terminal |
| `ps aux`                   | Show a broad list of running processes      |
| `systemctl status SERVICE` | Show the status of a systemd service        |
| `journalctl`               | Read systemd journal logs                   |
| `journalctl -u SERVICE`    | Read logs for a specific service            |

Examples:

```
ps aux
systemctl status jenkins
journalctl -u jenkins
```

Some commands depend on systemd and may behave differently inside WSL.

## 6. Command status and basic operators

\| Command or symbol | Meaning | |---|---| | `echo $?` | Display the previous command's exit status | | `&&` | Run the next command if the previous succeeds | | `||` | Run the next command if the previous fails | | `;` | Run commands sequentially regardless of success | | `\|` | Pass one command's standard output into another command |

Examples:

```
mkdir demo && cd demo
mkdir demo || echo "Creation failed"
pwd; date
```

Exit status `0` normally means success; a non-zero status indicates failure.

# Module 2 — Navigation, Files and Directories

## 1. Linux directory structure

| Path             | Meaning                                          |
| ---------------- | ------------------------------------------------ |
| `/`              | Root of the entire filesystem                    |
| `/home`          | Usually contains regular users' home directories |
| `/root`          | Root user's home directory                       |
| `/etc`           | System and application configuration             |
| `/var/log`       | Common location for system and application logs  |
| `/tmp`           | Temporary files                                  |
| `/opt`           | Often used for optional or third-party software  |
| `/usr/local/bin` | Common location for locally installed commands   |

## 2. Navigate directories

| Command    | Meaning                                               |
| ---------- | ----------------------------------------------------- |
| `pwd`      | Print the current directory                           |
| `ls`       | List directory contents                               |
| `ls -l`    | List detailed file information                        |
| `ls -la`   | Include hidden files and detailed information         |
| `cd /path` | Move to a specified directory                         |
| `cd ..`    | Move up one directory                                 |
| `cd ~`     | Move to your home directory                           |
| `cd -`     | Return to the previous directory                      |
| `tree`     | Display directories in a tree structure, if installed |

Examples:

```
pwd
ls -la
cd ~/linux-practice
cd ..
cd -
```

Path symbols:

- `.` — current directory.
- `..` — parent directory.
- `~` — current user's home directory.
- `/` at the beginning of a path — an absolute path from the filesystem root.

## 3. Create files and directories

| Command                        | Meaning                                                            |
| ------------------------------ | ------------------------------------------------------------------ |
| `mkdir test`                   | Create a directory                                                 |
| `mkdir -p a/b/c`               | Create parent directories as needed                                |
| `touch notes.txt`              | Create an empty file if it doesn't exist, or update its timestamps |
| `printf "Hello\n" > notes.txt` | Write text to a file, replacing existing contents                  |
| `nano notes.txt`               | Open a file in the Nano text editor                                |

Examples:

```
mkdir -p ~/linux-practice
cd ~/linux-practice
touch notes.txt
nano notes.txt
```

In Nano, use `Ctrl+O` to save, `Enter` to confirm the filename, and `Ctrl+X` to exit.

## 4. Copy, move and rename

| Command                 | Meaning                          |
| ----------------------- | -------------------------------- |
| `cp a.txt b.txt`        | Copy a file                      |
| `cp -r folder1 folder2` | Copy a directory recursively     |
| `mv old.txt new.txt`    | Rename a file                    |
| `mv file.txt /tmp/`     | Move a file to another directory |

Examples:

```
cp notes.txt notes-backup.txt
cp -r project project-backup
mv old.txt new.txt
```

## 5. Delete files and directories

| Command              | Meaning                                         |
| -------------------- | ----------------------------------------------- |
| `rm notes.txt`       | Delete a file                                   |
| `rmdir empty-folder` | Remove an empty directory                       |
| `rm -r folder`       | Delete a directory and its contents recursively |

Warning: `rm` generally does not send files to a recycle bin. Double-check the target before running a deletion command, especially with `rm -r`.

## 6. Wildcards

| Symbol  | Meaning                          | Example            |
| ------- | -------------------------------- | ------------------ |
| `*`     | Match zero or more characters    | `ls *.txt`         |
| `?`     | Match one character              | `ls file?.txt`     |
| `[abc]` | Match one character from the set | `ls file[123].txt` |

Example:

```
ls *.log
cp *.txt backup/
```

Wildcards are expanded by the shell before the command runs.

## 7. Links

| Command                 | Meaning                                                |
| ----------------------- | ------------------------------------------------------ |
| `ln -s target linkname` | Create a symbolic (soft) link                          |
| `ln target linkname`    | Create a hard link, subject to filesystem restrictions |

Examples:

```
ln -s /var/log/app.log app-log-link
ln original.txt hardlink.txt
```

A symbolic link points to a path and can cross filesystems. A hard link refers to the same underlying file data and generally cannot cross filesystems.

# Module 3 — File Viewing, Searching and Output

## 1. View files

| Command                | Meaning                                                   |
| ---------------------- | --------------------------------------------------------- |
| `cat file.txt`         | Display the entire file                                   |
| `less file.txt`        | Open a scrollable file viewer                             |
| `more file.txt`        | Display output one screen at a time                       |
| `head file.txt`        | Display the first 10 lines by default                     |
| `head -n 5 file.txt`   | Display the first 5 lines                                 |
| `tail file.txt`        | Display the last 10 lines by default                      |
| `tail -n 20 file.txt`  | Display the last 20 lines                                 |
| `tail -f app.log`      | Follow new lines added to a log                           |
| `tail -F app.log`      | Follow a log while retrying if it is rotated or recreated |
| `file app.log`         | Identify a file's type                                    |
| `diff old.txt new.txt` | Show differences between two files                        |

Useful `less` keyboard shortcuts:

| Key      | Meaning            |
| -------- | ------------------ |
| `Space`  | Next page          |
| `b`      | Previous page      |
| `/ERROR` | Search for `ERROR` |
| `n`      | Next search match  |
| `q`      | Quit               |

Stop `tail -f` or `tail -F` using `Ctrl+C`.

## 2. Search text with `grep`

| Command                   | Meaning                                                   |
| ------------------------- | --------------------------------------------------------- |
| `grep "ERROR" app.log`    | Display lines containing `ERROR`                          |
| `grep -i "error" app.log` | Search without case sensitivity                           |
| `grep -n "ERROR" app.log` | Include matching line numbers                             |
| `grep -c "ERROR" app.log` | Count matching lines                                      |
| `grep -v "INFO" app.log`  | Display lines that don't match `INFO`                     |
| `grep -r "ERROR" .`       | Search recursively through files in the current directory |
| `grep "ERROR" *.log`      | Search matching `.log` files in the current directory     |

Example:

```
grep -n "ERROR" app.log
```

This is a particularly useful command for troubleshooting application logs.

## 3. Count with `wc`

| Command                          | Meaning                             |
| -------------------------------- | ----------------------------------- |
| `wc -l file.txt`                 | Count lines                         |
| `wc -w file.txt`                 | Count words                         |
| `wc -c file.txt`                 | Count bytes                         |
| `cat app.log \\| wc -l`          | Count lines received through a pipe |
| `grep "ERROR" app.log \\| wc -l` | Count matching error lines          |

## 4. Write and append output

| Symbol or command | Meaning                                         |
| ----------------- | ----------------------------------------------- |
| `>`               | Redirect output, replacing the destination file |
| `>>`              | Append output to a file                         |
| `<`               | Take standard input from a file                 |
| `tee file.txt`    | Display standard input and write it to a file   |
| `tee -a file.txt` | Display standard input and append it to a file  |

Examples:

```
echo "Hello" > result.txt
echo "Welcome" >> result.txt
wc -l < app.log
ls -l | tee files.txt
```

Check your saved output with:

```
cat result.txt
```

## 5. Standard streams and error redirection

Linux uses three standard streams:

| Stream          | Number | Meaning        |
| --------------- | ------ | -------------- |
| Standard input  | `0`    | Input          |
| Standard output | `1`    | Normal output  |
| Standard error  | `2`    | Error messages |

| Command                                    | Meaning                                        |
| ------------------------------------------ | ---------------------------------------------- |
| `ls missing.txt 2> errors.txt`             | Save errors to a file, replacing its contents  |
| `ls missing.txt 2>> errors.txt`            | Append errors to a file                        |
| `ls app.log missing.txt > output.txt 2>&1` | Save normal output and errors in the same file |
| `ls missing.txt 2>/dev/null`               | Discard error messages                         |

Order matters: `> output.txt 2>&1` first redirects standard output, then sends standard error to the same destination.

## 6. Combine commands

| Operator | Meaning                                            | Example                           |
| -------- | -------------------------------------------------- | --------------------------------- |
| `\\|`    | Pass standard output to the next command           | `cat app.log \\| wc -l`           |
| `&&`     | Run the next command only if the previous succeeds | `mkdir test && cd test`           |
| `\\|\\|` | Run the next command only if the previous fails    | `mkdir test \\|\\| echo "Failed"` |
| `;`      | Run commands sequentially regardless of success    | `pwd; date`                       |
| `$?`     | Read the previous command's exit status            | `ls app.log; echo $?`             |

Examples of exit status:

- `0` — success.
- Non-zero — failure.

Remember: check `$?` immediately after the command whose result you want to inspect.

# Module 4 — Finding Files and Processing Text

## 1. `find` — Locate files and directories

| Command                                                   | Meaning                                      |
| --------------------------------------------------------- | -------------------------------------------- |
| `find . -name "*.log"`                                    | Find names ending in `.log`                  |
| `find . -type f`                                          | Find regular files                           |
| `find . -type d`                                          | Find directories                             |
| `find . -type f -name "*.txt"`                            | Find regular `.txt` files                    |
| `find . -iname "APP.LOG"`                                 | Match a name without case sensitivity        |
| `find . -type f -empty`                                   | Find empty files                             |
| `find . -type f -size +1M`                                | Find files larger than 1 MiB                 |
| `find . -type f -mtime -1`                                | Find files modified within the last 24 hours |
| `find . -type f -ls`                                      | Display details for found files              |
| `find . -type f -name "*.log" -exec grep -n "ERROR" {} +` | Search matching log files for `ERROR`        |

`-exec` runs another command on found files. The `{}` represents the found paths, and `+` passes multiple paths to the command when possible.

## 2. `sort` — Arrange lines

| Command                       | Meaning                      |
| ----------------------------- | ---------------------------- |
| `sort names.txt`              | Sort lines alphabetically    |
| `sort -r names.txt`           | Reverse sort order           |
| `sort -n numbers.txt`         | Sort numerically             |
| `sort -u names.txt`           | Sort and output unique lines |
| `sort names.txt > sorted.txt` | Save sorted output           |

For numeric values, use `sort -n`; ordinary `sort` compares text by default.

## 3. `uniq` — Handle duplicates

| Command                      | Meaning                         |
| ---------------------------- | ------------------------------- |
| `uniq names.txt`             | Remove adjacent duplicate lines |
| `uniq -c names.txt`          | Count adjacent duplicate groups |
| `uniq -d names.txt`          | Show adjacent duplicate lines   |
| `sort names.txt \\| uniq`    | Sort and remove duplicates      |
| `sort names.txt \\| uniq -c` | Count occurrences of each line  |

Example:

```
sort names.txt | uniq -c
```

This is useful for counting repeated log messages or event types.

## 4. `cut` — Extract fields or characters

| Command                           | Meaning                                 |
| --------------------------------- | --------------------------------------- |
| `cut -d ',' -f 1 employees.csv`   | Extract the first comma-separated field |
| `cut -d ',' -f 2 employees.csv`   | Extract the second field                |
| `cut -d ',' -f 1,3 employees.csv` | Extract fields 1 and 3                  |
| `cut -c 1-4 file.txt`             | Extract characters 1–4                  |
| `cut -d ':' -f 1 /etc/passwd`     | Extract the first colon-separated field |

- `-d` — delimiter.
- `-f` — field number.
- `-c` — character positions.

## 5. `awk` — Process columns and perform calculations

| Command                                           | Meaning                                        |
| ------------------------------------------------- | ---------------------------------------------- |
| `awk '{print $0}' file.txt`                       | Print the complete line                        |
| `awk '{print $1}' file.txt`                       | Print the first field                          |
| `awk '{print $1, $3}' file.txt`                   | Print fields 1 and 3                           |
| `awk '$3 > 45000' employees.txt`                  | Print rows where field 3 is greater than 45000 |
| `awk '{sum += $3} END {print sum}' employees.txt` | Add field 3 and print the total                |
| `awk -F ',' '{print $1}' employees.csv`           | Split fields using commas                      |
| `awk '{print $1}' app.log \\| sort \\| uniq -c`   | Count occurrences of each first field          |

Important symbols:

- `$1` — first field.
- `$2` — second field.
- `$0` — entire line.
- `END` — execute after all input has been processed.
- `-F` — specify the field separator.

## 6. `sed` — Transform text

| Command                                       | Meaning                                        |
| --------------------------------------------- | ---------------------------------------------- |
| `sed 's/ERROR/WARNING/' app.log`              | Replace the first match on each line in output |
| `sed 's/ERROR/WARNING/g' app.log`             | Replace all matches on each line               |
| `sed '/INFO/d' app.log`                       | Omit lines containing `INFO` from output       |
| `sed -n '1,5p' app.log`                       | Display lines 1–5                              |
| `sed 's/ERROR/ALERT/g' app.log > updated.log` | Save transformed output in another file        |
| `sed -i 's/old/new/g' file.txt`               | Modify the file in place                       |

`-i` edits the original file, so use it carefully. Without `-i`, `sed` normally displays transformed output without changing the source file.

## 7. `xargs` — Convert input into command arguments

| Command                                               | Meaning                                    |
| ----------------------------------------------------- | ------------------------------------------ |
| `printf "apple\nbanana\n" \\| xargs echo`             | Pass both values to `echo`                 |
| `printf "one\ntwo\n" \\| xargs -I {} echo "Item: {}"` | Run the command with each item substituted |
| `find . -name "*.log" -print0 \\| xargs -0 file`      | Identify file types safely                 |
| `find . -name "*.txt" -print0 \\| xargs -0 wc -l`     | Count lines in found text files            |

- `-I {}` — use `{}` as a placeholder for an input item.
- `-print0` and `-0` — safely separate filenames, including those containing spaces.

Be careful when using `xargs` with commands that delete or modify files.

# Final revision: all four modules at a glance

Module 1 — Linux fundamentals

`whoami`, `id`, `uname`, `date`, `sudo`, `man`, `env`, `export`, `ps`, `systemctl`, `journalctl`, `$?`, `&&`, `||`

Module 2 — Files and directories

`pwd`, `ls`, `cd`, `mkdir`, `touch`, `nano`, `cp`, `mv`, `rm`, `rmdir`, wildcards, `ln`, `tree`

Module 3 — Viewing and output

`cat`, `less`, `more`, `head`, `tail`, `grep`, `wc`, `file`, `diff`, `tee`, pipes, redirection

Module 4 — Finding and processing text

`find`, `sort`, `uniq`, `cut`, `awk`, `sed`, `xargs`

## The most important DevOps command combinations

```
# Search errors
grep -n "ERROR" app.log

# Count errors
grep "ERROR" app.log | wc -l

# Count log levels
awk '{print $1}' app.log | sort | uniq -c

# Find errors across log files
find . -type f -name "*.log" -exec grep -n "ERROR" {} +

# Save matching errors
grep "ERROR" app.log > errors.txt

# Follow live logs
tail -f app.log
```

Study advice: Don't try to memorize every option at once. First, practise each command on files in `~/linux-practice`. Then practise the combined commands until you can explain what each part does. These four modules establish a solid Linux foundation for junior DevOps work.
