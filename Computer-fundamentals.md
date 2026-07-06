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

## RAM vs storage (HDD/SSD) & how a program loads into memory

**What it is:**
Storage (HDD/SSD) is where files live *permanently* — it survives shutdown but
is slower. RAM is fast, *temporary* memory for whatever is running right now,
and it's wiped when the power goes off.

Here's the part that ties it together — how a program actually runs:
1. A program sits on disk as a file (e.g. `chrome.exe`) — just data, doing nothing.
2. When you launch it, the operating system copies it from disk into RAM.
3. Once it's in RAM, the CPU can execute its instructions fast.
4. A program that's been loaded into memory and is running is called a **process**.

So: on disk = a sleeping program; in RAM = a running process.

**How I practiced it:**
- Windows: opened Task Manager → Processes tab, and watched a program appear as
  a running process the moment I launched it, then disappear when I closed it.
- Linux: ran `ps aux` to list running processes, and `free -h` to see how much
  RAM was in use before vs after opening an app.

**Real-world example:**
A game takes a few seconds to "load" — that pause is the OS copying it from the
slower disk into fast RAM. Once loaded, it runs smoothly because the CPU is now
reading from RAM, not the disk.

**Red team use:**
Attackers exploit the fact that a running process lives in RAM. "Fileless
malware" runs entirely in memory and never writes itself to disk, specifically
to avoid antivirus, which mostly scans files on disk.

**Blue team use:**
Because the useful evidence is in RAM and vanishes on reboot, responders capture
a machine's memory *before* powering it off. Memory forensics then reveals
running processes, hidden or injected code, and network connections — the things
memory-only attacks try to hide. Rule of thumb I picked up: "if it's not on disk,
look in memory."

---

## Credentials in memory — the LSASS concept

**What it is:**
LSASS (Local Security Authority Subsystem Service) is a core Windows process
that handles logins. While a user is signed in, it holds credential material in
its memory. This is a concrete example of the bigger idea I keep running into:
sensitive data lives in RAM while it's being used, not just on disk.

**Why it's a target (concept only):**
Because LSASS concentrates credential material in one place in memory, it's a
known target for credential theft. MITRE ATT&CK catalogues this as T1003.001
(OS Credential Dumping: LSASS Memory). I'm noting the concept and the technique
ID — not the how-to — because what I actually want is to understand and detect
this, not perform it.

**The defensive side (what caught my interest):**
Since this is so common, defenders build detection around it:
- monitoring for unusual processes trying to read LSASS's memory
- hardening LSASS with extra Windows protections
- reviewing security event logs for the access patterns credential theft leaves behind

Being able to *spot* credential dumping in logs feels like a real analyst skill,
and it connects straight back to why memory matters in the first place.

## Fetch–Decode–Execute — what the CPU is *actually* doing

Everyone says "the CPU runs instructions." Fine — but what does that mean, step
by step? The CPU runs a tiny loop, billions of times a second, and it only ever
does three things:

1. **Fetch** — go to memory and grab the next instruction.
2. **Decode** — figure out what that instruction means ("add these two numbers",
   "move this value", "jump somewhere else").
3. **Execute** — actually do it, and store the result.

Then it repeats. Forever. That loop is literally all a computer does — everything
else (games, browsers, malware) is just billions of these three steps.

The pieces involved:
- The **Program Counter (PC)** is a pointer that holds the address of the next
  instruction. After each step it moves forward — unless an instruction tells it
  to *jump* somewhere else.
- Instructions and data both live in RAM; the CPU keeps fetching from wherever
  the PC points.

**Why this matters (the part that makes it click for security):**
If an attacker can change what the Program Counter points to, they can make the
CPU "execute" data of their choosing instead of the real program. That single
idea — hijacking the flow of execution — is the beating heart of memory-corruption
exploits (the buffer-overflow family). I'm not going deep on exploitation here,
but knowing that "control the PC = control the CPU" is *why* those attacks exist
made the whole topic suddenly feel important instead of abstract.

Analogy that made it stick: the CPU is someone following a recipe card by card.
Fetch = pick up the next card. Decode = read what it says. Execute = do it. The
Program Counter is your finger tracking which card is next. An exploit is
someone slipping a fake card into the stack while your finger keeps moving.

---

## 32-bit vs 64-bit — what the number actually refers to

This isn't about speed like I assumed. It's mostly about **how big a number the
CPU can handle in one go**, and the biggest consequence is **how much memory it
can address.**

- A **32-bit** CPU/OS works with 32-bit addresses. The largest number 32 bits can
  represent is about 4.3 billion — so it can only address ~**4 GB of RAM**. Even
  if you install 16 GB, a 32-bit system can't "see" past ~4 GB.
- A **64-bit** system uses 64-bit addresses, which can count to a number so
  enormous the RAM limit is effectively irrelevant for now.

That 4 GB wall is the real, practical reason the whole world moved to 64-bit.

**How I verified it myself:**
    Windows:  Settings → System → About → "System type"
    Linux:    uname -m      ( x86_64 = 64-bit,  i686/i386 = 32-bit )
    Linux:    lscpu         ( look at "Architecture" and "CPU op-mode(s)" )

**Why it matters practically:**
- Software has to match: a 32-bit program can run on a 64-bit system, but not the
  reverse. This bites you constantly when installing tools — grabbing the wrong
  build is a classic "why won't this run?" moment.
- In security tooling you're forever choosing x86 (32-bit) vs x64 (64-bit)
  versions of things, and picking wrong just fails silently or crashes. Now I know
  *why* the choice exists instead of guessing.

---

## Endianness — the order bytes are stored in (awareness)

When a number is too big for one byte, it gets split across several bytes — and
there are two conventions for what order to store them in. That's endianness.

Take the number `0x12345678` (four bytes: 12 34 56 78):
- **Big-endian:** stored 12 34 56 78 — the "big" end first, the way we write it.
- **Little-endian:** stored 78 56 34 12 — the "little" (smallest) end first,
  looking reversed to human eyes.

Most computers you'll touch (Intel/AMD, i.e. x86) are **little-endian**, so bytes
look backwards in memory. Networks, by convention, use big-endian ("network byte
order").

**Why I bothered to note this (awareness level):**
The first time you stare at a hex dump in a tool like Wireshark or a hex editor
and a value looks *backwards*, endianness is the answer — you're not misreading
it, the machine just stored it little-end first. Knowing the word saves you an
hour of confusion. That's all I need for now; the deep detail can wait until it
actually comes up.

Analogy: same phone number, two habits for writing it down. Big-endian writes it
left-to-right like you'd say it; little-endian jots the last digits first. Same
number — you just have to know which habit you're reading.

