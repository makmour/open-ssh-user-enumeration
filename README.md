# OpenSSH Username Enumeration Script (CVE-2018-15473)

This script checks for the OpenSSH 7.7 (and prior) username enumeration vulnerability (CVE-2018-15473).
It sends a malformed authentication packet and interprets the SSH server’s response to identify valid usernames.

---

## original code:
https://www.exploit-db.com/download/45233

## Updates

1. Python Compatibility
Converted all print statements to Python 3 syntax `(print("..."))`.
Replaced `map(str.strip, f.readlines())` with a list comprehension for clarity and compatibility.

2. Paramiko 3.x+ Compatibility
Replaced:
```
paramiko.auth_handler.AuthHandler._handler_table[...]
```
with:
```
from paramiko.auth_handler import AuthHandler
client_table = AuthHandler._client_handler_table.fget(AuthHandler)
```
This avoids `TypeError: 'property' object is not subscriptable`.

4. Replaced direct patching:
`handler_table[MSG_SERVICE_ACCEPT] = malform_packet`
with
`client_table[paramiko.common.MSG_SERVICE_ACCEPT] = malform_packet`

5. RSA Key Generation Optimization
Avoided repeated generation of 1024-bit RSA keys (slow and insecure).
Introduced a cached 2048-bit RSAKey for testing.

6. Logging & Output Fixes
Removed reliance on args.outputFile being mandatory.
Added fallback to sys.stdout if --outputFile is not provided.

7. Minor fixes
Disabled Paramiko's noisy internal logging.
Replaced deprecated or redundant exception-handling patterns.
Applied consistent spacing/indentation (converted all tabs to 4 spaces).

## Requirements

- Python 3.6+
- Paramiko (tested with v3.4.0+)

Install dependencies:
```bash
pip3 install -r requirements.txt
```

---

## Usage

### Basic
```bash
python3 open-ssh-ue.py <hostname> --userList wordlist.txt
```

### Full Example
```bash
python3 open-ssh-ue.py hostname \
  --userList wordlist.txt \
  --threads 10 \
  --outputFile results.json \
  --outputFormat json
```

---

## Arguments

### Positional
- `hostname`: The target SSH server IP or domain.

### Optional
- `--port`: SSH port (default is `22`)
- `--threads`: Number of concurrent threads (default is `5`)
- `--userList`: Path to a username list file (one username per line)
- `--username`: Test a single username
- `--outputFile`: Path to save results (optional; prints to terminal if omitted)
- `--outputFormat`: Output format: `list`, `json`, or `csv` (default: `list`)

---

## Output Formats

- `list`: Plain text per-username result
- `json`: Structured list of valid/invalid usernames
- `csv`: Comma-separated values

---

## Legal Disclaimer

Use this tool **only on systems you own or have explicit permission to test**.
Unauthorized use is illegal and unethical.

---

## Reference

- [CVE-2018-15473 – OpenSSH Username Enumeration](https://nvd.nist.gov/vuln/detail/CVE-2018-15473)
