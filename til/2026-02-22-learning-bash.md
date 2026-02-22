# 📝 TIL — Bash Essentials, Concepts, and Examples
*A ready-to-commit, GitHub‑friendly summary of everything I practiced today.*

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

Notes

>&2 sends the usage message to stderr (best practice for errors/help).
exit 1 indicates a non‑zero, error exit code.

➕ 2) Arithmetic in Bash — Adding Two Numbers
Example Script

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

Quick tips

Arithmetic expansion: (( ... )) or $(( ... ))
Positional args: $1, $2, …
