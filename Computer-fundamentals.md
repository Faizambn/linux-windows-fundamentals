# Computer Fundamentals

My Module 1 notes, in my own words. For each topic I include:
what it is, how I practiced it, a real-world example, and how it's
used on both the red (offensive) and blue (defensive) side.

---

## 1. CPU, cores, registers, and cache

**What it is:**
The CPU is the brain of the computer — it executes instructions very fast.
Cores are individual workers inside it. Registers are tiny ultra-fast slots
holding what the CPU is using right now. Cache is small fast memory that keeps
recently-used data close so the CPU avoids the slower trip to RAM.

Speed order (fast → slow): registers → cache → RAM → disk.

**How I practiced it:**
- Opened Task Manager (Windows) → Performance tab → watched CPU cores spike
  while opening apps.
- On Linux, ran `lscpu` to see cores/threads, and `htop` to watch per-core load.

**Real-world example:**
When my laptop lags with many browser tabs open, that's the CPU and memory
under pressure — work is queuing up for a limited number of cores.

**Red team use:**
Attackers know CPU behavior gives them away — a crypto-mining payload pins the
CPU at 100%, so stealthy malware deliberately throttles itself to stay quiet.

**Blue team use:**
Defenders watch for abnormal CPU spikes as an indicator of compromise (e.g. a
process suddenly maxing a core can signal a miner or a runaway malicious script).

---

## 2. RAM vs storage (memory vs disk)

**What it is:**
RAM (memory) is fast, temporary storage for programs that are *currently
running*. It's wiped when the computer powers off. Disk (HDD/SSD) is slower but
*permanent* — it keeps files after shutdown.

**How I practiced it:**
- Task Manager → Memory graph while opening/closing programs.
- Linux: `free -h` to see RAM usage, `df -h` to see disk usage.

**Real-world example:**
An unsaved document lives in RAM — that's why a power cut loses it, but a saved
file survives because it's written to disk.

**Red team use:**
Passwords and encryption keys sit in RAM while in use. Attackers dump memory
(e.g. reading the LSASS process on Windows) to steal credentials that were never
written to disk.

**Blue team use:**
Memory forensics is a core investigation skill. Tools like Volatility (Module 7)
capture RAM to find hidden processes, injected code, and credentials — evidence
that vanishes on reboot, so responders grab memory *before* powering a machine off.

---

## 3. Data representation (binary, hex, Base64, encoding)

**What it is:**
Computers store everything as binary (0s and 1s). Hexadecimal (hex) is a shorter
way to write binary. Base64 and URL-encoding are ways to represent data as text
so it survives being sent over things like web requests.

Key point: **encoding is NOT encryption.** Base64 hides nothing — it's easily
reversed. It just repackages data.

**How I practiced it:**
- Used CyberChef (cyberchef.io) to convert text ↔ binary ↔ hex ↔ Base64 and
  watch how the same data looks in each form.
- Decoded a Base64 string by hand to prove it's reversible.

**Real-world example:**
Email attachments and images inside web pages are often Base64 — that long
gibberish string is just a file turned into text.

**Red team use:**
Attackers Base64-encode malicious commands to slip them past simple filters
(e.g. `powershell -enc <base64>`), because the payload doesn't "look" malicious
at a glance.

**Blue team use:**
Analysts learn to spot and decode Base64 in logs — a Base64 blob in a PowerShell
command line is a classic red flag, and decoding it reveals what the attacker
actually tried to run.

---

## Status
🚧 Actively learning — more topics added as I go.
