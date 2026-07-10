> 🧪 **See it live:** [Virtualization & Lab Setup — interactive version](https://faizambn.github.io/linux-windows-fundamentals/interactive/virtualization-lab-setup.html)
>
> Hypervisor-type toggle, VM anatomy, a network-mode reachability explorer, and a snapshot
> revert demo are on the interactive page. These notes are the quick-reference layer.

# Virtualization & Lab Setup — security cheat-sheet

You can't learn security on real systems (illegal + dangerous). Virtualization runs whole
computers *inside* your computer, so you can detonate malware and run exploits in a sealed
sandbox you can reset in seconds. This is where all hands-on practice happens.

---

## 1. Virtualization & hypervisors

A **hypervisor** splits one physical machine into multiple **virtual machines (VMs)**, each running its own OS. Your real OS = **host**; each VM = **guest**.

| Type | Sits on | Examples | You'll use it for |
|------|---------|----------|-------------------|
| **Type 1** (bare-metal) | Directly on hardware | ESXi, Hyper-V, Proxmox | Data centres & cloud |
| **Type 2** (hosted) | On top of your normal OS | **VirtualBox** (free), VMware Workstation | ⭐ Learning — installs like any app |

**Try it:** install VirtualBox (free), download a Kali Linux `.ova`, and import it.

---

## 2. Anatomy of a VM

A VM is just **virtual hardware** presented to a guest OS — and it lives as files on disk.

| Part | What it is |
|------|-----------|
| **vCPU** | A share of your real CPU cores |
| **Virtual RAM** | Memory carved from the host (the main limit — 16GB host runs only so many VMs) |
| **Virtual disk** (`.vdi`/`.vmdk`) | A file that the VM treats as a whole hard drive — copy = clone |
| **Virtual NIC** | A virtual network card; its **mode** decides what the VM can reach |

### Containers vs VMs

| | VM | Container (Docker) |
|--|----|--------------------|
| Isolation | Strong (own kernel) | Weaker (shares host kernel) |
| Weight | Heavy (GBs) | Light (starts in seconds) |
| Best for | Whole systems, **malware analysis** | Running apps/services fast |

| Attack | Note |
|--------|------|
| **VM escape** | Malware breaking out of a guest onto the host — rare but real; why isolation matters |

---

## 3. Network modes ⭐ (the part everyone gets wrong)

The VM's network mode is the most important lab setting. Set in VirtualBox: **Settings → Network → Attached to:**

| Mode | VM can reach | Use for |
|------|--------------|---------|
| **NAT** | Internet only (hidden behind host) | Downloading updates/tools safely |
| **Bridged** | Internet + your LAN + host (own IP on your network) | Acting as a real network device. ⚠ **Never for malware** |
| **Host-only** | The host only (no internet, no LAN) | Controlled host↔VM channel |
| **Internal** | Other VMs only (not even the host) | ⭐ The safest malware lab |

**🚨 Golden rule:** for malware analysis use **Internal** or **Host-only** so it's sealed from the internet and your real LAN. Bridged puts an infected VM straight onto your home network.

---

## 4. Building a lab + snapshots

**Starter lab:** an **attacker VM** (Kali Linux) + one or more **victim VMs** (a Windows box, or intentionally-vulnerable **Metasploitable**), all on one **Internal Network** so they reach each other but nothing else.

**Snapshots — the superpower:**

| Step | Action |
|------|--------|
| 1 | 📸 **Take a snapshot** (saves the VM's exact clean state) |
| 2 | 💥 Detonate malware / run the exploit — study the damage |
| 3 | ⏪ **Revert** — the VM is instantly pristine again, as if nothing ran |

This makes safe, repeatable malware analysis possible — and needs **no internet**, perfect for a local lab.

| No powerful PC? | Alternative |
|-----------------|-------------|
| Cloud ranges | **TryHackMe**, **Hack The Box** — ready-made labs in the browser |

---

## The one-line takeaway
Virtualization gives you a disposable computer to break safely. Use **Type 2** (VirtualBox) to
build a Kali-vs-victim lab on an **isolated Internal network**, and always **snapshot before
you detonate** so you can revert to clean in one click.

---
🚧 Actively learning — grows as I go.
