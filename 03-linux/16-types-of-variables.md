## Question  
What are the different types of variables in Linux?

### 📝 Short Explanation  
This checks whether you understand the difference between **shell (local) variables** and **environment variables**, and how scope works — a common source of "why isn't my variable available in the script/child process?" confusion.

## ✅ Answer  
Broadly there are two types:
- **Local (shell) variables** — exist only in the **current shell**, not passed to child processes.
- **Environment variables** — **exported** so they're inherited by **child processes**.

There are also **special/built-in shell variables** (like `$?`, `$$`, `$1`, `$#`) used in scripting.

### 📘 Detailed Explanation  

#### 1. 🏠 Local (shell) variables
Defined in the current shell; **not** inherited by child processes.
```bash
name="Anshul"
echo "$name"      # works in this shell
bash -c 'echo "$name"'   # empty — child process didn't inherit it
```

#### 2. 🌍 Environment variables
Created with `export`; **inherited** by any child process (scripts, programs).
```bash
export APP_ENV="prod"
bash -c 'echo "$APP_ENV"'   # prints prod
```
Common ones: `PATH`, `HOME`, `USER`, `SHELL`. View them with `env` or `printenv`.

#### 3. ⚙️ Special / built-in variables (scripting)
| Variable | Meaning |
|---|---|
| `$0` | Script name |
| `$1`, `$2`… | Positional arguments |
| `$#` | Number of arguments |
| `$@` / `$*` | All arguments |
| `$?` | Exit status of the last command |
| `$$` | PID of the current shell |
| `$!` | PID of the last background command |

```bash
cp file.txt /backup/ && echo "exit code: $?"   # $? = 0 on success
```

#### 🔑 Local vs Environment (the key distinction)

| | Local variable | Environment variable |
|---|---|---|
| Created by | `var=value` | `export var=value` |
| Scope | Current shell only | Current shell + child processes |
| Seen with | `set` | `env` / `printenv` |

---

### 🧠 Real-world Example

> A script kept failing because a variable set in the terminal wasn't visible inside it. The variable was **local** — I hadn't `export`ed it, so the child shell running the script never received it. Adding `export` fixed it. For permanent values I add the `export` line to `~/.bashrc` or `/etc/environment`.

> Summary:  
> Two main types: **local/shell** variables (current shell only) and **environment** variables (`export`ed, inherited by children). Plus **special variables** (`$?`, `$$`, `$1`, `$#`) for scripting. If a variable isn't visible in a child process, it likely needs to be **exported**.
