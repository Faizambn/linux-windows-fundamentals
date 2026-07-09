> 🖥️ **See it live:** [Operating System Concepts — interactive version](https://faizambn.github.io/linux-windows-fundamentals/interactive/operating-system-concepts.html)
>
> Concepts are on the interactive page. These notes are the quick-reference security layer.

# Operating System Concepts — security cheat-sheet

An OS is a set of **boundaries** (program↔hardware, program↔program, trusted↔untrusted).
Security = defending those boundaries. That's the whole topic in one line.

---

## 1. Kernel vs User Space

| | |
|---|---|
| **In one line** | Kernel mode (ring 0) can touch anything; user mode (ring 3) is fenced in. Apps ask the kernel via system calls. |
| **Real-world** | A browser can't format your disk directly — it must ask the OS. The fence is why one bad app can't wreck the machine. |

| Attack | Defense | Key tool |
|--------|---------|----------|
| Climb from user → kernel; hide as a **rootkit** below the OS | Least privilege · patching · driver signing · watch the boundary | **EDR** |

**EDR** = a sensor (partly in the kernel) that watches low-level behaviour — process
creation, driver loads, one process reading another's memory — and flags malicious
*sequences* (e.g. Word → PowerShell → loads driver). Can isolate & kill on detection.

**Try it:** `lsmod` (Linux) · `driverquery` (Windows) — lists code running in your kernel.

---

## 2. Processes vs Threads (+ states)

| Term | Meaning |
|------|---------|
| **Process** | A running program with its **own private memory** |
| **Thread** | A line of execution inside a process; threads **share** that memory |
| **States** | New → Ready → Running → Waiting → Terminated (the OS shuffles these as programs share the CPU) |

| Attack | Defense | Key tool |
|--------|---------|----------|
| **Process injection** — hide a malicious thread inside a *trusted* process (e.g. a browser) | Detect one process creating a thread/memory inside another | **Sysmon** |

**Real-world:** malware injects into `explorer.exe` so its activity wears a trusted name.
**Sysmon** = free Windows tool logging process creation (with command line + parent) and
injection signs, so a SIEM can alert.

**Try it:** `ps -eLf` (Linux) · Task Manager → Details (Windows).

---

## 3. Virtual Memory & Paging *(concept)*

| | |
|---|---|
| **In one line** | Every process *thinks* it owns all memory; the OS maps its **virtual pages** → real **RAM frames** (or disk if RAM is full). |
| **Page fault** | "That page isn't in RAM yet" — OS fetches it. Normal; a storm of them (thrashing) = RAM exhausted. |
| **Why it's security** | The mapping stops one process reading another's memory — an **isolation boundary**. |

| Attack | Defense |
|--------|---------|
| Memory-corruption exploits (e.g. buffer overflow) need to know *where* things are in memory | **ASLR** randomises memory locations so addresses can't be predicted · non-executable memory stops data running as code |

**Real-world:** you have 30 apps open with "more RAM in use" than you physically have —
that's paging quietly swapping to disk behind the scenes.

**Try it:** watch RAM + swap in `htop` (Linux) or Task Manager → Performance.

---

## 4. System Calls & Drivers *(concept)*

| Term | Meaning |
|------|---------|
| **System call** | The guarded request a program makes to cross into the kernel (open a file, send a packet) |
| **Driver** | Specialised kernel-mode code that operates one specific piece of hardware |

| Attack | Defense | Key tool |
|--------|---------|----------|
| **BYOVD** — load a *signed but vulnerable* driver, abuse its bug to run kernel code | Block known-vulnerable drivers · require signing · monitor syscall patterns | **strace** |

**Real-world:** a file request travels App → system call → kernel → disk driver → hardware,
and back. Drivers are prized targets because they run in the kernel.
**strace** = shows every system call a program makes live — X-ray vision into what a
binary actually asks the kernel to do.

**Try it:** `strace -e trace=open,read cat file.txt` (Linux).

---

## The one-line takeaway
An OS is boundaries; attackers cross them, defenders harden **and watch** them. Notice the
same tools keep reappearing (EDR, Sysmon) — because *watching boundaries* is the daily job.

---
🚧 Actively learning — grows as I go.
