# 📝 TIL — Bash Essentials, Concepts, and Examples
---

## 📌 1) Validating Script Arguments (`$#`, `$0`)
**Concepts**
- `$#` → number of positional arguments passed to the script  
- `$0` → the script name as invoked  
- `[[ $# -ne 2 ]]` → “if the number of args is **not equal to 2**”

**Example**
```bash
if [[ $# -ne 2 ]]; then
  echo "Usage: $0 <num1> <num2>" >&2
  exit 1
fi
```

**Notes**

>&2 sends the usage message to stderr (best practice for errors/help).
exit 1 indicates a non‑zero, error exit code.

*➕ 2) Arithmetic in Bash — Adding Two Numbers*

Example Script
```
#!/usr/bin/env bash
set -euo pipefail

if [[ $# -ne 2 ]]; then
  echo "Usage: $0 <num1> <num2>" >&2
  exit 1
fi

num1="$1"
num2="$2"

# Optional: simple integer validation
if ! [[ "$num1" =~ ^-?[0-9]+$ && "$num2" =~ ^-?[0-9]+$ ]]; then
  echo "Error: both arguments must be integers." >&2
  exit 1
fi

sum=$(( num1 + num2 ))
echo "Sum is: $sum"
```

> Quick tips

Arithmetic expansion: `(( ... )) or $(( ... ))`
Positional args: ** $1, $2, …**

**📁 3) Checking File Existence Using ls and $?**

***Why $??***
$? holds the exit status of the last command (0 = success; non‑zero = failure).
Pattern with strict mode

#!/usr/bin/env bash
set -euo pipefail

file="${1:-}"

Temporarily allow failure so we can read $?
set +e
output=$(ls -l -- "$file" 2>/dev/null)
status=$?
set -e

if [[ $status -eq 0 ]]; then
  # The 5th field of 'ls -l' is file size in bytes
  size=$(awk '{print $5}' <<<"$output")
  echo "Size: ${size} bytes"
else
  echo "Error: file '$file' not found." >&2
  exit 1
fi

Why set +e?
With set -e, a failing command aborts the script. We disable it briefly to let ls fail gracefully, capture $?, then restore set -e.
Alternatives (keep -e on):

1) Guard with 'if'
if output=$(ls -l -- "$file" 2>/dev/null); then
  size=$(awk '{print $5}' <<<"$output")
  echo "Size: ${size} bytes"
else
  echo "Error: file not found." >&2
fi

2) Use '|| true'
output=$(ls -l -- "$file" 2>/dev/null) || true
status=$?

##**🔁 4) Background Jobs and $! (PID of Last Background Process)**
Concepts

& runs a command in the background.
$! expands to the PID of the most recently started background job.

Example
```
#!/usr/bin/env bash
set -euo pipefail

sleep 60 &
echo "Started background job with PID: $!"
```

##**📊 5) Log Analysis with awk, sort, uniq**

a) Extract All Unique IPs (first field in common access logs)
```
awk '{print $1}' access.log | sort -u
```
> Explanation

awk '{print $1}' → print the first whitespace‑separated field (usually client IP)
sort -u → sort and keep unique values only

b) Most Frequent IP Address

awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -n 1

**Pipeline logic**

awk '{print $1}' → extract IPs
sort → group identical IPs together
uniq -c → count duplicates
sort -nr → sort numerically, reversed (highest count first)
head -n 1 → show the top (most frequent) IP

Print only the IP (without count)
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -n 1 | awk '{print $2}'

6) find -exec and {} + vs \;
Basic per‑file execution

```find /path -type f -exec command {} \;```

Runs command once per file (slower for many files).

**Batched (faster) execution**

```find /path -type f -exec command {} +
```

Appends many pathnames at once to command, repeating as needed (similar to xargs batching).
Safer for unusual filenames than plain xargs (no need for -print0/-0).

##**🆚 7) Why -exec Instead of xargs (and when to use xargs)**

Use -exec … {} + when

You want safe handling of filenames (spaces, newlines, quotes).
You prefer simple, self‑contained find expressions.

Use xargs when

You want parallelism (xargs -P 4).
The target command is stdin‑friendly (e.g., xargs cat > out.txt).

Safe xargs pattern
```
find . -type f -print0 | xargs -0 grep -l "error"
```

##**🧷 8) Strict Mode: set -e, set +e, set -o pipefail, set -u**

set -e → exit on a command failure
set +e → temporarily disable that behavior (to inspect $?)
set -o pipefail → a pipeline fails if any command in it fails
set -u → error on use of unset variables

```
Example
#!/usr/bin/env bash
set -euo pipefail   # strict mode

set +e
ls missing.file >/dev/null 2>&1
status=$?
set -e

if [[ $status -ne 0 ]]; then
  echo "File not found (as expected) — handled safely."
fi
```

##**🧩 9) Understanding and Setting PS1 (Your Bash Prompt)**

Examples (temporary in current shell)
```
PS1="\W$ "      # just the current directory
PS1="\u@\h$ "   # username@hostname
PS1="$ "        # minimal
```
**Make it permanent (Ubuntu)**

Edit ~/.bashrc
Add your export PS1='...' at the end (after the Ubuntu color prompt block)
source ~/.bashrc

**Common issues**

You’re not in bash (e.g., zsh/fish)
PROMPT_COMMAND or later lines overwrite PS1
You’re in a root shell (sudo -i) and editing the wrong rc file

##**🔀 10) Redirection & File Descriptors**
**Common patterns**
```
cmd > out.txt          # stdout to file (overwrite)
cmd >> out.txt         # stdout append
cmd 2> err.txt         # stderr to file
cmd &> all.txt         # stdout and stderr to the same file (bash)
cmd > out.txt 2>&1     # redirect stderr to stdout target
```
<img width="377" height="283" alt="image" src="https://github.com/user-attachments/assets/d5354c2d-b071-4ec0-ac2c-3360902935b0" />

##**🧰 12) Everyday Bash Commands (Quick Reminders)**

Basics
```
ls -lAh
cd /path
pwd
mkdir -p dir/subdir
touch file
rm -rf dir_or_file
cp -r src dst
mv old new
```
