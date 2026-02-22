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
