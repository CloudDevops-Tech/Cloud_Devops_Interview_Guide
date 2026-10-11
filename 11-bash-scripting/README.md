# 📜 TOPIC: BASH SCRIPTING

> Strict mode, special variables, conditionals, quoting, redirection, text processing, loops, functions, arrays, error handling, practical automation scripts and cron troubleshooting.

<br>

---

### Q1. What are shebang, exit codes, and `set -euo pipefail`? Why do production scripts use them?

#### 📌 Short Explanation
Tests scripting fundamentals and defensive programming habits.

#### ✅ Answer
The **shebang** (`#!/bin/bash`) tells the OS which interpreter to use. Every command returns an **exit code** (`0` = success, non-zero = failure). `set -euo pipefail` makes a script **fail fast** instead of silently continuing after errors.

#### 📖 Detailed Explanation

**🔹 Basics**
```bash
#!/bin/bash
echo "Hello"
echo "Last command exit code: $?"
```
```bash
chmod +x script.sh
./script.sh
```

**🔹 Strict mode**
```bash
#!/bin/bash
set -euo pipefail
```

| Option | Effect |
|--------|--------|
| `-e` | Exit immediately if a command fails |
| `-u` | Treat **unset variables** as errors |
| `-o pipefail` | A pipeline fails if **any** command in it fails (not just the last) |

**🔹 Why `pipefail` matters**
```bash
set -e
false | true; echo "continues"      # without pipefail, pipeline succeeds
set -o pipefail
false | true; echo "never reached"  # pipeline fails → script exits
```

**🔹 Exit codes to remember**

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | General error |
| `2` | Misuse of shell command |
| `126` | Command found but not executable |
| `127` | Command not found |
| `130` | Terminated by Ctrl+C |

> ⚠️ `set -e` has edge cases (e.g., inside `if`, `&&`/`||` chains); for critical logic, check exit codes explicitly.

<br>

---

### Q2. What is the difference between `$@`, `$*`, `$#`, `$0`, `$$`, `$!`, and `$?`?

#### 📌 Short Explanation
A very common bash question testing special parameters.

#### ✅ Answer
These are special shell variables for arguments and process information. The key trap is `"$@"` vs `"$*"`.

#### 📖 Detailed Explanation

| Variable | Meaning |
|----------|---------|
| `$0` | Script name |
| `$1 … $9` | Positional arguments |
| `$#` | Number of arguments |
| `$@` | All arguments (each preserved as a separate word when quoted) |
| `$*` | All arguments (joined into **one string** when quoted) |
| `$?` | Exit status of the last command |
| `$$` | PID of the current shell/script |
| `$!` | PID of the last background process |

**🔹 `"$@"` vs `"$*"`**
```bash
#!/bin/bash
# Run: ./demo.sh "hello world" foo
for arg in "$@"; do echo "[$arg]"; done
# [hello world]
# [foo]

for arg in "$*"; do echo "[$arg]"; done
# [hello world foo]
```

> **Key takeaway:** Almost always use **`"$@"`** to pass arguments through safely.

**🔹 Argument validation**
```bash
if [ $# -lt 2 ]; then
  echo "Usage: $0 <source> <dest>" >&2
  exit 1
fi
```

<br>

---

### Q3. What is the difference between `[ ]`, `[[ ]]`, and `(( ))` in Bash?

#### 📌 Short Explanation
Tests knowledge of conditional constructs and common portability and quoting pitfalls.

#### ✅ Answer
- **`[ ]`** — POSIX `test` command (portable, requires careful quoting)
- **`[[ ]]`** — Bash keyword with safer, richer features (pattern matching, regex, `&&`/`||`)
- **`(( ))`** — arithmetic evaluation

#### 📖 Detailed Explanation

| Feature | `[ ]` | `[[ ]]` | `(( ))` |
|---------|-------|---------|---------|
| **Portable (sh)** | ✅ | ❌ Bash/Zsh/Ksh | ❌ Bash |
| **Word splitting on unquoted vars** | ⚠️ Yes (must quote) | ✅ Safe | n/a |
| **Pattern match (`== *.txt`)** | ❌ | ✅ | ❌ |
| **Regex (`=~`)** | ❌ | ✅ | ❌ |
| **Logical operators** | `-a`, `-o` (discouraged) | `&&`, `\|\|` | `&&`, `\|\|` |
| **Numeric compare** | `-eq -lt -gt` | `-eq -lt -gt` | `== < >` |

**🔹 Examples**
```bash
[ -f /etc/passwd ] && echo "exists"

if [[ $name == dev-* ]]; then echo "dev env"; fi
if [[ $email =~ ^[a-z]+@example\.com$ ]]; then echo "valid"; fi

if (( count > 10 )); then echo "big"; fi
(( total = a + b ))
```

**🔹 Common file tests:** `-f` file, `-d` directory, `-e` exists, `-r/-w/-x` permissions, `-s` non-empty, `-z` empty string, `-n` non-empty string.

<br>

---

### Q4. What is the difference between single quotes, double quotes, and backticks? Why should you always quote variables?

#### 📌 Short Explanation
Quoting bugs cause real production incidents (e.g., `rm -rf $dir/` with an empty variable).

#### ✅ Answer
- **Single quotes `'...'`** — literal, no expansion
- **Double quotes `"..."`** — expand variables and commands, prevent word splitting/globbing
- **Backticks `` `cmd` ``** — old command substitution; prefer **`$(cmd)`**

#### 📖 Detailed Explanation

```bash
name="DevOps"
echo 'Hello $name'        # Hello $name
echo "Hello $name"        # Hello DevOps
echo "Today is $(date +%F)"
```

**🔹 Why quote?**
```bash
file="my report.txt"
rm $file          # ❌ tries to delete "my" and "report.txt"
rm "$file"        # ✅ deletes "my report.txt"
```

**🔹 `$(...)` vs backticks:** `$(...)` nests cleanly and is easier to read.
```bash
echo "$(basename "$(dirname "$path")")"
```

**🔹 Parameter expansion (very handy)**

| Syntax | Meaning |
|--------|---------|
| `${var:-default}` | Use `default` if unset/empty |
| `${var:?message}` | Error and exit if unset |
| `${#var}` | Length |
| `${var%.txt}` | Remove suffix |
| `${var#*/}` | Remove shortest prefix match |
| `${var^^}` / `${var,,}` | Upper / lower case |

> 💡 Dangerous pattern protection: `rm -rf "${DIR:?DIR is not set}/"*`

<br>

---

### Q5. Explain redirection and pipes: `>`, `>>`, `2>`, `2>&1`, `&>`, `/dev/null`, and `|`. What are stdout and stderr?

#### 📌 Short Explanation
Essential for logging, cron jobs, and troubleshooting.

#### ✅ Answer
Every process has **stdin (0)**, **stdout (1)**, and **stderr (2)**. Redirection controls where they go; pipes connect one command's stdout to another's stdin.

#### 📖 Detailed Explanation

| Syntax | Meaning |
|--------|---------|
| `cmd > file` | Redirect stdout (overwrite) |
| `cmd >> file` | Redirect stdout (append) |
| `cmd 2> err.log` | Redirect stderr |
| `cmd > out.log 2>&1` | stdout **and** stderr to the same file |
| `cmd &> all.log` | Bash shorthand for the above |
| `cmd > /dev/null 2>&1` | Discard all output |
| `cmd1 \| cmd2` | Pipe stdout of cmd1 into cmd2 |
| `cmd \| tee file` | Show on screen **and** write to file |
| `cmd < file` | Read stdin from file |

**🔹 Order matters**
```bash
cmd > out.log 2>&1     # ✅ both go to out.log
cmd 2>&1 > out.log     # ❌ stderr goes to the terminal, only stdout to the file
```

**🔹 Here-doc**
```bash
cat <<EOT > /etc/app.conf
port=8080
env=${ENV}
EOT
```

**🔹 Logging pattern for scripts**
```bash
exec > >(tee -a /var/log/myscript.log) 2>&1
echo "Started at $(date)"
```

<br>

---

### Q6. What is the difference between running `./script.sh`, `bash script.sh`, and `source script.sh`?

#### 📌 Short Explanation
A classic question that checks whether you understand subshells and environment.

#### ✅ Answer
- `./script.sh` and `bash script.sh` run in a **new child shell** (changes don't affect the parent).
- `source script.sh` (or `. script.sh`) runs **in the current shell**, so variables, `cd`, and `export` persist.

#### 📖 Detailed Explanation

| Method | Needs `+x`? | Runs in | Variables persist in parent? |
|--------|-------------|---------|------------------------------|
| `./script.sh` | ✅ Yes | Child shell (uses shebang) | ❌ No |
| `bash script.sh` | ❌ No | Child shell | ❌ No |
| `source script.sh` | ❌ No | **Current shell** | ✅ Yes |

**🔹 Example**
```bash
# env.sh
export APP_ENV=prod
cd /opt/app
```
```bash
bash env.sh;   echo $APP_ENV     # (empty) – child shell exited
source env.sh; echo $APP_ENV     # prod – and current directory changed
```

**🔹 Use cases:** `source` for loading environment files (`source .env`), activating Python venvs (`source venv/bin/activate`).

**🔹 Subshell trap**
```bash
count=0
echo "a" | while read -r x; do count=$((count+1)); done
echo $count     # 0  → pipeline loop ran in a subshell
```
Fix: use process substitution: `while read -r x; do ...; done < <(echo "a")`.

<br>

---

### Q7. What is the difference between `grep`, `sed`, and `awk`? When do you use each?

#### 📌 Short Explanation
Text-processing tools are central to Linux/DevOps work.

#### ✅ Answer
- **`grep`** — search/filter lines matching a pattern
- **`sed`** — stream editor for substitution, deletion, and line edits
- **`awk`** — column/field-based processing with logic and calculations

#### 📖 Detailed Explanation

| Tool | Best for | Example |
|------|----------|---------|
| `grep` | Finding lines | `grep -i "error" app.log` |
| `sed` | Find/replace, delete lines | `sed -i 's/8080/9090/g' app.conf` |
| `awk` | Columns, aggregation, reports | `awk '{print $1, $9}' access.log` |

**🔹 grep**
```bash
grep -rn "TODO" ./src            # recursive with line numbers
grep -v "DEBUG" app.log          # invert match
grep -E "ERROR|WARN" app.log     # extended regex
grep -c "500" access.log         # count matches
```

**🔹 sed**
```bash
sed -n '10,20p' file             # print lines 10–20
sed '/^#/d' file                 # delete comment lines
sed -i.bak 's/old/new/g' file    # in-place edit with backup
```

**🔹 awk**
```bash
awk -F: '{print $1}' /etc/passwd                 # usernames
awk '{sum+=$3} END {print "Total:", sum}' data   # sum a column
awk '$9 == 500 {print $1}' access.log            # IPs with 500 status
df -h | awk '$5+0 > 80 {print $6, $5}'           # filesystems above 80%
```

**🔹 Other useful tools:** `cut`, `sort`, `uniq`, `tr`, `xargs`, `jq` (JSON), `find`.

<br>

---

### Q8. How do you loop over files and read a file line by line safely? Scenario: process a CSV of servers.

#### 📌 Short Explanation
Looping and file-reading patterns are used in almost every automation script.

#### ✅ Answer
Use `for` for lists/globs and `while IFS= read -r line` for reading files line by line (preserves spaces and backslashes).

#### 📖 Detailed Explanation

**🔹 for loops**
```bash
for f in /var/log/*.log; do
  echo "Processing $f"
done

for i in {1..5}; do echo "$i"; done

for ((i=0; i<5; i++)); do echo "$i"; done
```

**🔹 while loop to read lines**
```bash
while IFS= read -r line; do
  echo "Line: $line"
done < input.txt
```

**🔹 Scenario: servers.csv (`name,ip,env`)**
```bash
#!/bin/bash
set -euo pipefail

while IFS=, read -r name ip env; do
  [[ "$name" == "name" ]] && continue          # skip header
  if ping -c1 -W1 "$ip" &>/dev/null; then
    echo "$name ($env) - UP"
  else
    echo "$name ($env) - DOWN"
  fi
done < servers.csv
```

**🔹 Common mistakes**
- ❌ `for line in $(cat file)` — splits on whitespace, breaks lines containing spaces
- ❌ Forgetting `-r` → backslashes are interpreted
- ❌ Piping into `while` and expecting variable changes to persist (subshell issue)

**🔹 Loop controls:** `break`, `continue`; `until` loops until a condition becomes true.

<br>

---

### Q9. What are functions and local variables in Bash? What is the difference between `return` and `exit`?

#### 📌 Short Explanation
Tests modular scripting and variable scope.

#### ✅ Answer
Functions group reusable logic. Variables are **global by default**, so use **`local`** inside functions. **`return`** exits the function (with a status); **`exit`** terminates the entire script.

#### 📖 Detailed Explanation

```bash
#!/bin/bash
set -euo pipefail

log() {
  local level="$1"; shift
  echo "$(date '+%F %T') [$level] $*"
}

check_disk() {
  local mount="$1" threshold="${2:-80}"
  local usage
  usage=$(df --output=pcent "$mount" | tail -1 | tr -dc '0-9')
  if (( usage >= threshold )); then
    return 1
  fi
  return 0
}

if ! check_disk / 85; then
  log WARN "Root filesystem is almost full"
  exit 2
fi
log INFO "Disk OK"
```

| Keyword | Effect |
|---------|--------|
| `return N` | Leaves the **function** with status N (0–255) |
| `exit N` | Terminates the **script** with status N |
| `local var` | Variable scoped to the function |

**🔹 Returning data:** functions return only a numeric status; **print** values and capture with `$(func)`.
```bash
get_ip() { hostname -I | awk '{print $1}'; }
ip=$(get_ip)
```

<br>

---

### Q10. How do arrays and associative arrays work in Bash?

#### 📌 Short Explanation
Tests data-structure usage in scripts, e.g., looping over services or environment maps.

#### ✅ Answer
Bash supports **indexed arrays** and **associative arrays** (Bash 4+, declared with `declare -A`).

#### 📖 Detailed Explanation

**🔹 Indexed array**
```bash
services=(nginx sshd docker)

echo "${services[0]}"          # nginx
echo "${#services[@]}"         # 3
services+=(cron)               # append

for s in "${services[@]}"; do
  systemctl is-active --quiet "$s" && echo "$s: running" || echo "$s: stopped"
done
```

**🔹 Associative array**
```bash
declare -A ports=( [web]=80 [api]=8080 [db]=5432 )

for name in "${!ports[@]}"; do
  echo "$name -> ${ports[$name]}"
done

[[ -v ports[web] ]] && echo "web is defined"
```

**🔹 Useful operations**

| Syntax | Meaning |
|--------|---------|
| `${arr[@]}` | All elements |
| `${!arr[@]}` | All indexes / keys |
| `${#arr[@]}` | Number of elements |
| `${arr[@]:1:2}` | Slice |
| `unset arr[1]` | Remove element |

**🔹 Read command output into an array**
```bash
mapfile -t files < <(find /var/log -name "*.log" -mtime +7)
```

<br>

---

### Q11. How do you handle errors in Bash? Explain `trap`, cleanup, and `set -e` pitfalls.

#### 📌 Short Explanation
Reliable scripts clean up after themselves and report failures clearly.

#### ✅ Answer
Use **`trap`** to run cleanup/handlers on `EXIT`, `ERR`, or signals; check exit codes explicitly; and send errors to **stderr**.

#### 📖 Detailed Explanation

**🔹 Cleanup with `trap`**
```bash
#!/bin/bash
set -euo pipefail

TMP_DIR=$(mktemp -d)
cleanup() {
  rm -rf "$TMP_DIR"
  echo "Cleaned up $TMP_DIR"
}
trap cleanup EXIT                         # always runs, even on error/exit

trap 'echo "Error on line $LINENO" >&2' ERR
trap 'echo "Interrupted"; exit 130' INT TERM
```

**🔹 Explicit error checks**
```bash
if ! cp "$src" "$dest"; then
  echo "Copy failed" >&2
  exit 1
fi

command -v aws >/dev/null 2>&1 || { echo "aws CLI not installed" >&2; exit 1; }
```

**🔹 `set -e` pitfalls**
- Not triggered inside `if`, `while`, or on the left side of `&&`/`||`
- Doesn't catch failures within command substitution assigned with `local x=$(cmd)` (declare and assign separately)
- A failing command in a pipeline is hidden unless `pipefail` is set

**🔹 Retry pattern**
```bash
retry() {
  local n=0 max=5 delay=3
  until "$@"; do
    (( ++n >= max )) && { echo "Failed after $max attempts" >&2; return 1; }
    sleep "$delay"
  done
}
retry curl -fsS https://example.com/health
```

<br>

---

### Q12. Scenario: Write a script that alerts when disk usage exceeds 80%.

#### 📌 Short Explanation
A very common practical interview task combining `df`, `awk`, loops, and alerts.

#### ✅ Answer
Parse `df` output, compare each filesystem with a threshold, and send an email or webhook alert. Schedule it with cron.

#### 📖 Detailed Explanation

**🔹 `disk_alert.sh`**
```bash
#!/bin/bash
set -euo pipefail

THRESHOLD=${1:-80}
HOST=$(hostname)
ALERTS=""

while read -r usage mount; do
  usage=${usage%\%}
  if (( usage >= THRESHOLD )); then
    ALERTS+="Filesystem ${mount} is at ${usage}% on ${HOST}"$'\n'
  fi
done < <(df -hP -x tmpfs -x devtmpfs | awk 'NR>1 {print $5, $6}')

if [[ -n "$ALERTS" ]]; then
  echo "$ALERTS"
  echo "$ALERTS" | mail -s "Disk usage alert on ${HOST}" ops@example.com
  # Or Slack: curl -X POST -H 'Content-type: application/json' --data "{\"text\":\"$ALERTS\"}" "$SLACK_WEBHOOK"
fi
```

**🔹 Schedule (every 15 minutes)**
```bash
*/15 * * * * /opt/scripts/disk_alert.sh 80 >> /var/log/disk_alert.log 2>&1
```

**🔹 Next steps when alerted:** `du -xh / --max-depth=1 | sort -h | tail`, check large logs, `lsof +L1` for deleted-but-open files.

<br>

---

### Q13. Scenario: Write a backup script that archives a directory, uploads to S3, and keeps only the last 7 local backups.

#### 📌 Short Explanation
Combines variables, archiving, AWS CLI, rotation, error handling, and logging.

#### ✅ Answer
Create a timestamped `tar.gz`, upload with `aws s3 cp`, verify success, and delete older local backups.

#### 📖 Detailed Explanation

**🔹 `backup.sh`**
```bash
#!/bin/bash
set -euo pipefail

SRC_DIR="/var/www/app"
BACKUP_DIR="/backups"
BUCKET="s3://my-company-backups/app"
RETENTION=7
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
ARCHIVE="${BACKUP_DIR}/app_${TIMESTAMP}.tar.gz"

log() { echo "$(date '+%F %T') $*"; }
trap 'log "Backup FAILED"; exit 1' ERR

mkdir -p "$BACKUP_DIR"

log "Creating archive $ARCHIVE"
tar -czf "$ARCHIVE" -C "$(dirname "$SRC_DIR")" "$(basename "$SRC_DIR")"

log "Uploading to S3"
aws s3 cp "$ARCHIVE" "${BUCKET}/" --only-show-errors

log "Removing local backups beyond last ${RETENTION}"
ls -1t "${BACKUP_DIR}"/app_*.tar.gz | tail -n +$((RETENTION + 1)) | xargs -r rm -f

log "Backup completed successfully"
```

**🔹 Notes**
- Run as a **least-privilege IAM role** (instance profile), not with hardcoded keys
- Use **S3 lifecycle rules** for remote retention
- Verify with `tar -tzf` or compare checksums, and test **restores** regularly

<br>

---

### Q14. Scenario: Find the top 10 IP addresses hitting your web server from an access log.

#### 📌 Short Explanation
A favorite log-analysis question that tests pipelines.

#### ✅ Answer
Extract the IP column, sort, count unique occurrences, sort by count, and take the top 10.

#### 📖 Detailed Explanation

**🔹 One-liner**
```bash
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head -10
```

**🔹 How it works**

| Step | Purpose |
|------|---------|
| `awk '{print $1}'` | Extract the client IP (first field) |
| `sort` | Group identical IPs together (needed for `uniq`) |
| `uniq -c` | Count occurrences |
| `sort -nr` | Sort numerically, highest first |
| `head -10` | Top 10 |

**🔹 Variations**
```bash
# Top URLs returning 404
awk '$9 == 404 {print $7}' access.log | sort | uniq -c | sort -nr | head

# Requests per status code
awk '{print $9}' access.log | sort | uniq -c | sort -nr

# Requests in the last hour (log date format dependent)
grep "$(date -d '1 hour ago' '+%d/%b/%Y:%H')" access.log | wc -l

# Top IPs from a compressed log
zcat access.log.2.gz | awk '{print $1}' | sort | uniq -c | sort -nr | head
```

> 💡 Use `fail2ban`, WAF rate limiting, or Security Group/NACL deny rules to block abusive IPs.

<br>

---

### Q15. Scenario: How do you debug a Bash script that misbehaves? What is ShellCheck?

#### 📌 Short Explanation
Tests your debugging approach and tooling knowledge.

#### ✅ Answer
Use **`bash -x`** or `set -x` to trace execution, `bash -n` to check syntax, and **ShellCheck** to catch common bugs before running.

#### 📖 Detailed Explanation

**🔹 Debug modes**
```bash
bash -n script.sh        # syntax check only (no execution)
bash -x script.sh        # trace every command with expanded values
bash -v script.sh        # print lines as they are read
```

**🔹 Targeted tracing inside a script**
```bash
set -x     # start tracing
risky_function
set +x     # stop tracing
```

**🔹 Make traces readable**
```bash
export PS4='+ ${BASH_SOURCE}:${LINENO}: '
```

**🔹 ShellCheck**
```bash
shellcheck script.sh
```
Catches issues like unquoted variables, useless `cat`, wrong `[ ]` usage, and unsafe `rm`.

**🔹 Typical debugging checklist**
- Missing quotes around variables? · Wrong shebang or Windows line endings (`dos2unix`)? · `set -u` triggering on unset variables? · Relying on a different PATH or environment than expected?
- Add `echo`/`log` statements and check `$?` after critical commands
- Run **ShellCheck in CI** to prevent regressions

<br>

---

### Q16. Scenario: Ensure only one instance of a script runs at a time (prevent overlapping cron runs).

#### 📌 Short Explanation
Overlapping runs can corrupt data or exhaust resources. Tests knowledge of locking.

#### ✅ Answer
Use **`flock`** (preferred), or a lock file with PID checks and a `trap` to clean it up.

#### 📖 Detailed Explanation

**🔹 Using `flock` inside the script**
```bash
#!/bin/bash
exec 200>/var/lock/myjob.lock
flock -n 200 || { echo "Another instance is running. Exiting."; exit 1; }

# ... job logic ...
```

**🔹 Using `flock` directly in cron**
```bash
*/5 * * * * flock -n /var/lock/myjob.lock /opt/scripts/myjob.sh
```

**🔹 Lock file with PID check (portable)**
```bash
LOCK=/tmp/myjob.pid
if [[ -f "$LOCK" ]] && kill -0 "$(cat "$LOCK")" 2>/dev/null; then
  echo "Already running (PID $(cat "$LOCK"))"; exit 1
fi
echo $$ > "$LOCK"
trap 'rm -f "$LOCK"' EXIT
```

**🔹 Why `flock` is better:** the lock is released automatically by the kernel if the process dies, avoiding **stale lock files**.

<br>

---

### Q17. Scenario: A script works when run manually but fails from cron. Why?

#### 📌 Short Explanation
One of the most common real-world scripting problems.

#### ✅ Answer
Cron runs with a **minimal environment** (limited `PATH`, no profile, different working directory and shell). Use **absolute paths**, set the environment explicitly, and log output.

#### 📖 Detailed Explanation

**🔹 Cron syntax**
```text
┌───────── minute (0–59)
│ ┌─────── hour (0–23)
│ │ ┌───── day of month (1–31)
│ │ │ ┌─── month (1–12)
│ │ │ │ ┌─ day of week (0–7, Sun=0 or 7)
* * * * *  command
```

**🔹 Typical causes**

| Cause | Fix |
|-------|-----|
| **Limited `PATH`** (`/usr/bin:/bin`) | Use full paths or set `PATH=` at the top of the crontab/script |
| **Env vars not loaded** (`.bashrc`/`.bash_profile` not sourced) | `source` needed files in the script |
| **Relative paths / wrong working dir** | `cd /opt/app \|\| exit 1` or use absolute paths |
| **Different shell** (`/bin/sh`) | Set `SHELL=/bin/bash` in crontab |
| **Permissions / wrong user** | Check the crontab owner (`crontab -l -u user`) and file permissions |
| **No output visible** | Redirect: `>> /var/log/job.log 2>&1` |
| **`%` character in the command** | Escape as `\%` |
| **Script not executable / missing shebang** | `chmod +x`, add `#!/bin/bash` |

**🔹 Debug approach**
```bash
* * * * * env > /tmp/cron_env.txt          # inspect cron's environment
grep CRON /var/log/syslog                  # Debian/Ubuntu
journalctl -u cron                         # systemd
```

**🔹 cron vs at vs systemd timers**

| Tool | Use |
|------|-----|
| `cron` | Recurring schedules |
| `at` | One-time scheduled job |
| `systemd timer` | Modern alternative with logging, dependencies, and calendar events |

<br>

---

<br>

