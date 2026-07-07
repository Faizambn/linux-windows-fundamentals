> 🔢 **The concept, explained visually:** [interactive version](https://faizambn.github.io/linux-windows-fundamentals/data-representation.html)
>
> The interactive page teaches *what* these things are. These notes are the part
> that matters for a security career: how attackers turn each concept into a weapon,
> how defenders catch them, and how the key tools on both sides actually work.

# Data Representation — the security perspective

Everything a computer handles is, underneath, a number, and encoding is simply the
costume we dress that number in. This sounds academic until you realise that almost
every web attack is a variation of one trick: making a system misread a disguised
piece of data. Once you see representation as *disguise*, the attacks and defenses
below stop being separate facts and become the same idea repeating.

## Number bases

An attacker's first move against a filter is often to write a forbidden value in a
form the filter doesn't recognise. A blocklist watching for the text `127.0.0.1`
is trivially defeated by writing that same address as `0x7f000001` in hex or
`2130706433` as a single decimal number — the browser and the operating system
happily interpret all three as the same place, but the naive text-match never fires.
The defender's answer is *canonicalisation*: before you compare anything to a rule,
you first convert it to one standard form, so every disguise collapses back into the
value it was hiding.

The tool you learn this on is **CyberChef**. It's a free, browser-based workbench
where you build a "recipe" — a vertical stack of small operations like "From Hex" or
"From Base64" — and your input flows down through each step, transforming as it goes.
Its cleverness is the "Magic" operation: it tries many decoders against your data and
scores each result on how much it looks like meaningful text, so it can automatically
peel back layered disguises an analyst couldn't guess by eye. That's the whole job of
an analyst facing obfuscation, automated.

## Bits and bytes

This one is less about disguise and more about *volume*. When an attacker steals data,
that theft is measured in bytes leaving the network, so a patient attacker will trickle
data out slowly to stay beneath whatever size threshold might trigger an alarm. The
defender therefore watches not the content but the *quantity and shape* of outbound
traffic — an unusual surge of bytes to an unfamiliar destination is a classic sign of
exfiltration.

The mechanism behind that watching is **NetFlow** (and a SIEM that consumes it). NetFlow
doesn't record what's inside your packets — it records *metadata* about each conversation:
who talked to whom, on what port, for how long, and how many bytes and packets crossed.
A SIEM ingests millions of these flow records and lets the analyst ask questions like
"which internal host sent the most data outbound this week?" Because it's counting bytes
rather than reading payloads, it works even when the traffic is encrypted — which is why
volume-based detection is so durable.

## ASCII, Unicode and UTF-8

Because Unicode gives every character a unique number, two characters can look identical
to a human yet be completely different to a computer. The Cyrillic `а` (U+0430) is a
different number from the Latin `a` (U+0061) despite being visually the same, and an
attacker exploits this by registering a lookalike domain such as `аpple.com`. The victim
reads a trusted name and hands over their password. The defender's principle is to trust
the number, never the shape: browsers quietly convert such names into their `xn--`
punycode form to expose them, and security systems normalise text to one canonical form
before any comparison, so the impostor can't slip through a check for the real name.

The offensive-and-defensive tool worth knowing here is **dnstwist**. You give it your own
domain, and its permutation engine generates thousands of plausible lookalikes — character
swaps, common typos, alternate endings, and homoglyph substitutions like the Cyrillic
trick — then it queries DNS for each one to see which have actually been registered and
are live. A defender runs it to discover an impersonation domain *before* the phishing
campaign using it goes out; the same tool shows an attacker which disguises are still
available. Understanding it means understanding that this whole class of attack is
predictable enough to be enumerated in advance.

## Base64

Base64 turns any bytes into plain text, and attackers lean on it not to hide data from
machines but to hide intent from *humans* skimming a screen. A malicious command wrapped
as `powershell -enc <base64>` looks like meaningless noise at a glance, even though it
reverses instantly. Because it's encoding and not encryption, the defender's job is simply
to notice the blob and decode it — the difficulty is never the decoding, it's *capturing*
the command in the first place.

That capture is what **PowerShell script-block logging** does. When enabled, Windows records
the actual blocks of PowerShell code as they execute and writes them to the event log
(Event ID 4104) in their *decoded, deobfuscated* form. So even if an attacker delivered
their command Base64-encoded, the log shows the real, readable script that ultimately ran.
It's a perfect illustration of the defensive pattern: don't try to block every disguise at
the door — instead, log what actually executes, after the disguises have fallen away.

## URL encoding

URLs can't contain many characters directly, so each is written as `%` followed by its hex
value — a space becomes `%20`, a single quote `%27`, a `<` becomes `%3C`. Attackers use this
to smuggle attack characters past filters, and the sharpest version is *double encoding*:
the `%` itself can be encoded, so `<` becomes `%3C` and then `%253C`. A filter that blocks
the single form never sees the double form, yet the server may decode it back into a live
`<`. The defense is, once again, canonicalisation — fully decoding a request, repeatedly,
until it stops changing, *before* inspecting it.

The tool doing this at scale is a **Web Application Firewall (WAF)**. A WAF sits in front of
your web application and inspects every incoming HTTP request against a library of rules that
recognise attack patterns like SQL injection or cross-site scripting. Its effectiveness hinges
entirely on the normalisation step: it must decode all the layers of encoding first, because a
rule looking for a raw `<script>` is useless if the payload arrives as `%253Cscript%253E`. This
is exactly why double-encoding attacks exist — they target the gap between what the WAF decodes
and what the application ultimately interprets.

---

## The thread through all of it

Every defense above is the same move: **decode the disguise, reduce it to one standard form,
then compare.** That is a handful of lines of code — which is the honest reason scripting is a
core security skill rather than an optional one. The attacker automates disguising; the
defender automates un-disguising. Learning to write those small scripts is learning to do the
job.

🚧 Actively learning — grows as I go.
