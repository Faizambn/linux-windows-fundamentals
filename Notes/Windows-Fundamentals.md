> 🪟 **See it live:** [Windows Fundamentals — interactive version](https://faizambn.github.io/linux-windows-fundamentals/interactive/windows-fundamentals.html)
>
> Registry explorer, Event ID attack-replay, UAC demo, and a PowerShell/CMD terminal are on
> the interactive page. These notes are the quick-reference security layer.

# Windows Fundamentals — security cheat-sheet

Most companies run Windows, so most real attacks — and most defending — happen here.
The Registry, event logs, and how Windows handles users and privilege are core SOC skills.

---

## 1. The Registry ⭐ (persistence + forensics)

A hierarchical database of settings — like Linux's `/etc`, as one tree.
**Hives** (branches) → **keys** (folders) → **values** (settings).

| Hive / key | Holds | Why security cares |
|------------|-------|--------------------|
| `HKLM` | Machine-wide settings (needs admin) | Machine-level config & persistence |
| `HKLM\...\Run` | Programs auto-started for **all** users | ⭐ #1 malware persistence spot |
| `HKLM\SYSTEM\...\Services` | Service configuration | Attackers add/hijack a service to persist |
| `HKLM\SAM` | Local password **hashes** | ⭐ Windows' `/etc/shadow` — dump & crack |
| `HKCU` | Current-user settings (no admin) | Per-user persistence without admin |
| `HKCU\...\Run` | This user's auto-start programs | ⭐ persistence needing no admin |

**Try it:** `regedit` (GUI) · `reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Run"`

---

## 2. Event Logs & Event IDs ⭐ (the blue-team core)

Windows records events by number. Open with `eventvwr` → Windows Logs → Security.

| Event ID | Meaning | Watch for |
|----------|---------|-----------|
| **4624** | Successful logon | Odd time / source / account |
| **4625** | Failed logon | A **burst** = brute-force |
| **4688** | Process creation | Word → PowerShell = suspicious |
| **4672** | Special (admin) privileges assigned | A normal user getting these |
| **4720** | User account **created** | Unexpected = backdoor account |
| **1102** | Security log **cleared** | 🚨 attacker covering tracks |

**Attack story in logs:** 200× `4625` → `4624` → `4672` → `4720` → `1102` = brute-force → success → admin → backdoor user → wiped logs. Reading this is Tier-1 SOC work.

---

## 3. Users, groups, SIDs & UAC

| Concept | Meaning |
|---------|---------|
| **SID** | Unique ID for every account (e.g. `S-1-5-21-…-500`) |
| **RID** | The tail of the SID. **500** = built-in Administrator, **501** = Guest |
| **SYSTEM** (`S-1-5-18`) | The OS itself — the most powerful account |
| **UAC** | Even admins run with a *filtered* token until they explicitly elevate ("Run as Administrator") |

| Attack | Defense |
|--------|---------|
| Target RID-500 / SYSTEM · **UAC bypass** (elevate without the prompt) · **token theft** (steal another session's access) | Monitor `4672` elevation · least privilege · keep UAC on |

**Try it:** `whoami /user` (your SID) · `whoami /priv` (your privileges)

---

## 4. NTFS & file permissions (ACLs)

Windows uses **ACLs** (Access Control Lists) — richer than Linux rwx. `(F)`=full, `(M)`=modify, `(R)`=read.

| Command | Does |
|---------|------|
| `icacls file` | show a file's permissions |
| `icacls file /grant user:R` | grant read |

**Attack/Defense:** `Everyone:(F)` on a sensitive file = a misconfiguration attackers hunt; defenders audit ACLs on key folders.

---

## 5. Processes, services & scheduled tasks

| Command | Does |
|---------|------|
| `tasklist` / `Get-Process` | running processes (CMD / PowerShell) |
| `taskkill /PID n` | stop a process |
| `sc query` / `services.msc` | services |
| `schtasks /query` | scheduled tasks |

**Attack/Defense:** malware masquerades as `svchost.exe`, or persists via a **service** or **scheduled task** (e.g. run daily at 3am). Defenders baseline what should exist and flag new entries.

---

## 6. PowerShell & CMD

| Shell | Note |
|-------|------|
| **CMD** | The old command prompt (`dir`, `ipconfig`, `net user`) |
| **PowerShell** | Powerful, object-based (`Get-Process`, `Get-Service`); verb-noun cmdlets |

| Attack | Defense |
|--------|---------|
| **Living off the land** — use built-in PowerShell so nothing looks foreign; hide commands with `-enc` Base64 | **Script-block logging** (Event ID 4104) records the real, decoded commands |

**Try it:** `Get-Process` · `whoami` · `systeminfo`

---

## 7. Networking (Windows)

| Command | Shows |
|---------|-------|
| `ipconfig /all` | IP, gateway, DNS |
| `netstat -ano` | active connections + owning PID |
| `nslookup host` | DNS lookup |

**Attack/Defense:** `netstat -ano` reveals a connection to an attacker's server (C2); mapping a PID to a process is how you catch what's "phoning home."

---

## 8. Filesystem & key directories

Unlike Linux's single `/`, Windows uses **drive letters** (`C:\`, `D:\`). Windows ships in **editions** (Home / Pro / Enterprise) on the same core.

| Location | Holds | Why security cares |
|----------|-------|--------------------|
| `C:\Windows\System32` | Core OS programs & DLLs | ⭐ Malware fakes a System32 name (bogus `svchost.exe`) |
| `C:\Program Files` | Installed apps (admin-only to write) | A program running from here is more trustworthy |
| `C:\Users\<name>` | Personal files, browser data | Evidence lives here in an investigation |
| `…\AppData` | Per-user app data (hidden) | ⭐ #1 malware drop spot — writable without admin |
| `C:\Windows\Temp`, `%TEMP%` | Temp files | Common staging area for droppers |

**Try it:** `dir C:\Windows\System32` · `dir %APPDATA%`

---

## 9. Built-in security (Defender & Firewall)

| Tool | Command | Note |
|------|---------|------|
| **Microsoft Defender** | `Get-MpComputerStatus` | Built-in antivirus; attackers try to disable it |
| **Windows Firewall** | `netsh advfirewall show allprofiles` | Blocks/allows traffic per profile |

**Attack/Defense:** a common early attacker move is turning Defender/Firewall **off**; a defender's basic check is confirming both are **on** and healthy.

---

## 10. The defender's toolkit — Sysinternals ⭐

Microsoft's free **Sysinternals** suite is the essential Windows investigation kit:

| Tool | Does |
|------|------|
| **Process Explorer** | Super Task Manager — shows parent/child, signatures, what a process really is |
| **Autoruns** | Lists **everything** set to auto-start — the fastest way to spot persistence |
| **Procmon** | Records every file / registry / process action live |

If you learn one Windows toolset, learn this one. Get it free: `learn.microsoft.com/sysinternals`.

---

## The one-line takeaway
Windows security = the Registry (where persistence hides), the Event Log (where the story
is told), and identity/privilege (SIDs, UAC, tokens). Attackers persist and escalate;
defenders read the logs and hunt the odd entry.

---
🚧 Actively learning — grows as I go.
