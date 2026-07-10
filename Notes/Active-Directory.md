> 🗝️ **See it live:** [Active Directory — interactive version](https://faizambn.github.io/linux-windows-fundamentals/interactive/active-directory.html)
>
> Structure explorer, Kerberos ticket-flow, a BloodHound-style attack path, and an attacks/defenses
> breakdown are on the interactive page. These notes are the quick-reference security layer — every concept covered.

# Active Directory (intro) — security cheat-sheet

Active Directory (AD) is the identity system for most companies: it decides **who every
user is** and **what they can access**. Compromise it and you own the whole network — which
is why "get to Domain Admin" is the goal of nearly every enterprise attack.

---

## 1. Structure

| Term | Meaning |
|------|---------|
| **Forest** | Top-level container and the **true security boundary**; holds one or more domains |
| **Tree** | A group of domains sharing a namespace (e.g. `corp.com`, `eu.corp.com`) |
| **Domain** | A management + authentication boundary for users/computers/groups |
| **Domain Controller (DC)** | ⭐ The server running AD; authenticates logons; holds the database |
| **NTDS.dit** | ⭐ The DC's database file — stores **every** account's password hash |
| **Organizational Unit (OU)** | Folder grouping objects for management & policy |
| **SYSVOL** | A share on every DC holding Group Policy files (checked for stored secrets) |
| **Global Catalog** | Forest-wide index for fast searches across domains |

**Why it's the top target:** own a DC → read NTDS.dit → every hash → every account.

---

## 2. Objects & groups

| Object | Note |
|--------|------|
| **User** | A person's account (name, groups, password) |
| **Computer** | Every domain-joined machine has an account too (with a password) |
| **Group** | ⭐ A bundle of accounts used to grant permissions in bulk |

**Key privileged groups:** **Domain Users** (everyone), **Domain Admins** (full domain control), **Enterprise Admins** (full *forest* control).

**Group types/scopes** (intro level): security vs distribution groups; scopes are Domain Local, Global, Universal.

| Attack | Defense |
|--------|---------|
| Enumerate who's in privileged groups; get added to Domain Admins | Least privilege; review group membership; alert on changes to admin groups |

---

## 3. The plumbing: LDAP, DNS, GPO

| Piece | Does | Security angle |
|-------|------|----------------|
| **LDAP** | The protocol used to **query** AD | How attackers enumerate all users/groups/computers |
| **DNS** | How clients **find** Domain Controllers & services | AD literally won't work without it; SRV records point to DCs |
| **Group Policy (GPO)** | Central rules pushed to machines (passwords, software, settings) | Edit a GPO → push malware everywhere it applies; SYSVOL holds GPO files |
| **Trusts** | Let one domain/forest access another | An over-broad trust lets an attacker hop between domains |

---

## 4. Authentication: Kerberos & NTLM ⭐

**Kerberos** (the modern default) uses **tickets** so your password isn't sent everywhere:

| Piece | Meaning |
|-------|---------|
| **KDC** | The ticket office (runs on the DC) |
| **TGT** (Ticket-Granting Ticket) | Your "I'm authenticated" master pass, sealed with the **krbtgt** key |
| **Service Ticket (TGS)** | A day-pass for one specific resource |

**Flow:** prove yourself to the KDC → get a **TGT** → trade the TGT for a **Service Ticket** → present it to the service → access.

**NTLM** = the older challenge/response protocol, still around as fallback — where the password **hash** alone can authenticate (this is what enables Pass-the-Hash).

| Attack | Defense |
|--------|---------|
| **Kerberoasting**, **Golden Ticket**, Pass-the-Hash (NTLM) | Strong service-account passwords; disable NTLM where possible; watch Kerberos events **4768/4769** |

---

## 5. Common AD attacks

| Attack | How it works (short) | Tool | Defense |
|--------|----------------------|------|---------|
| **Kerberoasting** | Request a service ticket, crack it offline for the service account's password | Rubeus, Hashcat | Long/random service passwords (gMSA) |
| **Pass-the-Hash** | Reuse a stolen NTLM hash to log in — no plaintext needed | Mimikatz, Impacket | Disable NTLM; Credential Guard; tiering |
| **Golden Ticket** | Forge an unlimited TGT using the stolen `krbtgt` key = total control | Mimikatz | Protect DCs; rotate krbtgt (twice) |
| **DCSync** | Ask a DC to "replicate" and hand over every hash | Mimikatz, secretsdump | Restrict replication rights to real DCs |
| **LLMNR/NBT-NS poisoning** | Answer a machine's name broadcast, capture its sent creds | Responder | Disable LLMNR/NBT-NS via GPO; SMB signing |

---

## 6. Attack paths & the key tool — BloodHound

Attackers rarely jump straight to Domain Admin — they follow a **path**: `user → member of group → local admin on server → where a Domain Admin is logged in → steal token → Domain Admin`.

**BloodHound** collects AD data and draws it as a graph, then finds the **shortest path** from any user to Domain Admin. Attackers use it to plan; defenders use it to find and cut those paths first. This is *the* AD tool to know.

---

## 7. Defender's toolkit

| Defense | What it does |
|---------|--------------|
| **Tiering** | Keep admin accounts off ordinary workstations (limits token theft) |
| **LAPS** | Unique local-admin password per machine (stops one hash unlocking all) |
| **Disable NTLM** | Removes Pass-the-Hash where possible |
| **Microsoft Defender for Identity** | Watches DC traffic and flags Kerberoasting, DCSync, Golden Ticket, etc. live |
| **Event monitoring** | 4768/4769 (Kerberos), 4624/4625 (logons), 4720 (new user), changes to admin groups |

**Attacker tools recap:** BloodHound (map), Mimikatz (dump creds/tickets), Responder (poison & capture), Rubeus/Impacket (Kerberos abuse).

---

## The one-line takeaway
Active Directory is identity for the whole company. Attackers enumerate it (LDAP/BloodHound),
abuse authentication (Kerberos/NTLM), and follow permission chains to Domain Admin; defenders
harden the DCs, tier admin access, disable legacy protocols, and watch the tell-tale events.

---
🚧 Actively learning — grows as I go.
