> 🐧 **See it live:** [Linux Fundamentals — interactive version](https://faizambn.github.io/linux-windows-fundamentals/interactive/linux-fundamentals.html)
>
> Concepts + a live permissions calculator, terminal, and log-analysis builder are on the
> interactive page. These notes are the quick-reference security layer — covering every subtopic.

# Linux Fundamentals — security cheat-sheet

Linux runs servers, cloud, and nearly every security tool. Fluency in its filesystem,
permissions, and command line is the ground everything else stands on.

---

## 1. Distros & the shell

| Thing | Note |
|-------|------|
| **Debian** | The stable base many others are built on |
| **Ubuntu** | Debian-based; the most common beginner/server distro |
| **Kali** | Debian-based, pre-loaded with hacking/security tools — the pentester's distro |
| **Shell** | The program that reads your commands. **bash** is the default; **zsh** is a fancier alternative (Kali's default) |

**Security note:** Kali is for *your* attacking machine; you'll defend and investigate Ubuntu/Debian servers.

---

## 2. The filesystem (FHS)

Everything lives under one root `/` — no drive letters.

| Location | Holds | Why security cares |
|----------|-------|--------------------|
| `/etc` | System config | Secrets & misconfigs |
| `/etc/passwd` | User accounts (world-readable, no passwords) | Enumerate users; UID 0 = root |
| `/etc/shadow` | Password **hashes** (root-only) | The privesc prize — crack offline |
| `/home` | User files, `~/.ssh`, `.bash_history` | Rich pickings |
| `/var/log` | Logs (`auth.log`, `syslog`) | Heart of blue-team work |
| `/tmp` | Temp files, **world-writable** | Attacker staging area |
| `/bin`, `/usr/bin` | The actual commands | Trojaned binary = persistence |
| `/root` | Root user's home | If you can read it, you're probably root |

**Try it:** `ls /` · `cat /etc/passwd`

---

## 3. Navigation & file operations

`pwd` (where am I) · `cd` (change dir) · `ls -la` (list all + permissions) · `cat`/`less` (view) ·
`cp`/`mv`/`rm` (copy/move/delete) · `mkdir`/`touch` (make folder/file) · `find`/`locate` (search).

---

## 4. Permissions ⭐ (the #1 privesc surface)

Three groups — **owner / group / others** — each with **r(4) w(2) x(1)**.

| Number | Symbolic | Meaning |
|--------|----------|---------|
| `644` | `-rw-r--r--` | normal file |
| `755` | `-rwxr-xr-x` | script / directory |
| `600` | `-rw-------` | private (SSH key) |
| `777` | `-rwxrwxrwx` | ⚠ everyone everything — dangerous |
| `4755` | `-rwsr-xr-x` | ⚠ **SUID** — runs as owner (root!) |

| Command | Does |
|---------|------|
| `chmod 755 f` | set permissions (numeric or `chmod u+x f` symbolic) |
| `chown user:group f` | change who **owns** a file |
| `umask` | the default permissions new files are created with (e.g. 022) |

**Special bits:** **SUID** (runs as file's owner), **SGID** (as its group), **Sticky** (only owner can delete — used on `/tmp`, shows as `t`).

| Attack | Defense |
|--------|---------|
| Abuse world-writable or SUID-root files to become root | `find / -perm -4000` audits SUID files · least privilege · sane `umask` |

**Try it:** `chmod 600 secret.txt && ls -l secret.txt` → `-rw-------`

---

## 5. Users & groups

`whoami` (who am I) · `id` (UID/GID + groups; `sudo` group = can escalate) ·
`sudo -l` ⭐ (what you can run as root) · `su - user` (switch user).
Accounts in `/etc/passwd`, hashes in `/etc/shadow`.

**Attack/Defense:** loose `sudo -l` or a user in `sudo` group → root. Defenders tighten sudoers & watch it for changes.

---

## 6. Package management

| Command | Does |
|---------|------|
| `apt install/update/remove x` | high-level package manager (Debian/Ubuntu) |
| `dpkg -i file.deb` | install a single `.deb` package directly |
| `dpkg -l` | list installed packages |

**Security note:** outdated packages = known vulnerabilities; `apt update` is basic patching.

---

## 7. Processes & job control

| Command | Does |
|---------|------|
| `ps aux` | all processes + their user |
| `top` / `htop` | live monitor |
| `kill <pid>` | stop a process |
| `jobs` | list background jobs in this shell |
| `command &` | run a command in the background |
| `nohup command &` | keep it running after you log out |

**Attack/Defense:** malware hides as a process / uses `nohup` to persist; defenders baseline what should run.

---

## 8. Services (systemd) & scheduled tasks

| Command | Does |
|---------|------|
| `systemctl status/start/stop ssh` | manage a service |
| `systemctl enable ssh` | start it at boot |
| `journalctl -u ssh` | that service's logs |
| `crontab -l` / `crontab -e` | list / edit scheduled jobs |
| `/etc/crontab` | system-wide schedule |

**Attack/Defense:** a malicious **cron** job or systemd service is classic persistence; watch for new ones.

---

## 9. Text processing, pipes & redirection ⭐ (the SOC superpower)

**Pipe `|`** = send one command's output into the next.
**Redirection:** `>` write to file (overwrite) · `>>` append · `<` read from file · `2>&1` send errors along with output.

| Tool | Does |
|------|------|
| `grep "x"` | keep lines containing x |
| `awk '{print $N}'` | print field N |
| `sed 's/a/b/'` | find & replace |
| `cut -d: -f1` | slice by delimiter |
| `sort` / `uniq -c` | sort / count duplicates |
| `wc -l` | count lines |
| `head` / `tail` | first / last lines (`tail -f` = follow live) |

**Real-world (find a brute-force attack):**
