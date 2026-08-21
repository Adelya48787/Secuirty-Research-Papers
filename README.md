<div align="center">

# isolated-vm: From Type Confusion to Sandbox Escape and Potential RCE

**A technical write-up of `GHSA-864f-rcv7-6rh4`** — a critical type-confusion vulnerability in [`isolated-vm`](https://github.com/laverdet/isolated-vm) that lets sandboxed JavaScript corrupt host memory and potentially achieve remote code execution.

[![Severity: Critical](https://img.shields.io/badge/Severity-Critical-b23a2e?style=flat-square)](#)
[![Advisory](https://img.shields.io/badge/GHSA-864f--rcv7--6rh4-1B2A4A?style=flat-square)](https://github.com/laverdet/isolated-vm/security/advisories/GHSA-864f-rcv7-6rh4)
[![Affected](https://img.shields.io/badge/Affected-%E2%89%A4%207.0.0-b5751b?style=flat-square)](#affected-versions)
[![Fixed in](https://img.shields.io/badge/Fixed%20in-6.2.0%20%2F%207.0.1-2e7d32?style=flat-square)](#the-fix)
[![CVE](https://img.shields.io/badge/CVE-pending%20assignment-6b7280?style=flat-square)](#)

</div>

---

## TL;DR

| | |
|---|---|
| **Component** | [`isolated-vm`](https://www.npmjs.com/package/isolated-vm) — Node.js sandbox for untrusted JavaScript |
| **Root cause** | Type confusion via a **TOCTOU gap** in `ExternalCopy`'s handling of the `transferList` option |
| **Impact** | Host memory corruption → control-flow hijack → **guest-to-host sandbox escape** → potential RCE |
| **Trigger** | A single `ivm.Reference` — the minimum capability most hosts already grant a sandbox |
| **Fixed in** | `7.0.1` and `6.2.0` |
| **Action required** | **Upgrade immediately** if you run `isolated-vm ≤ 7.0.0` |

> [!WARNING]
> `isolated-vm` is downloaded nearly **1 million times a week** and is used as the sandboxing layer in several popular AI agent frameworks. If your application executes untrusted or LLM-generated JavaScript through it, treat this as a priority patch.

---

## Table of Contents

- [How isolated-vm Works](#how-isolated-vm-works)
- [The Role of ExternalCopy](#the-role-of-externalcopy)
- [Type Confusion](#type-confusion)
- [From Type Confusion to Memory Corruption](#from-type-confusion-to-memory-corruption)
- [From Memory Corruption to Control-Flow Manipulation](#from-memory-corruption-to-control-flow-manipulation)
- [Sandbox Escape and Potential RCE](#sandbox-escape-and-potential-rce)
- [The Fix](#the-fix)
- [Mitigations and Takeaways](#mitigations-and-takeaways)
- [References](#references)

---

## How isolated-vm Works

`isolated-vm` runs untrusted JavaScript inside a separate **V8 Isolate** rather than directly inside the main Node.js process. The untrusted code is meant to stay inside that isolated environment — with no direct access to host memory and no way to influence the host's execution flow.


The sandbox and host still need to exchange data from time to time — and that's where **`ExternalCopy`** comes in.



## Affected Versions

| Version range | Status |
|---|---|
| `≤ 7.0.0` | 🔴 Vulnerable |
| `6.2.0` | ✅ Patched (6.x line) |
| `7.0.1` | ✅ Patched |

## Mitigations and Takeaways

1. **Upgrade now.** Move to `isolated-vm@7.0.1` (or `6.2.0` on the 6.x line). The fix is small and directly closes both the type confusion and the underlying execution-during-copy issue.
2. **Treat every capability you share into a sandbox as an attack surface.** A single `Reference` was enough to reach the vulnerable constructor. Share the absolute minimum, and assume anything you expose can be turned back on you.
3. **Scrutinize the glue, not just the boundary.** A correct isolation model doesn't make a library safe on its own — the marshaling layer, especially native code handling attacker-controlled objects, deserves the same scrutiny as the primitive it wraps.
4. **Know your dependencies' real security posture with reachability.** Whether this bug is exploitable for you depends on whether untrusted code can reach `ExternalCopy` with attacker-influenced input. Software composition analysis with reachability answers that instead of guessing.

```bash
# Upgrade
npm install isolated-vm@^7.0.1
# or, on the 6.x line
npm install isolated-vm@^6.2.0
```

## References

- [The Hacker News — Isolated-vm Flaw Lets Sandboxed JavaScript Escape to Host for Potential RCE](https://thehackernews.com/2026/08/isolated-vm-flaw-lets-sandboxed.html)
- [Endor Labs — GHSA-864f-rcv7-6rh4: Critical Type Confusion Vulnerability in isolated-vm](https://www.endorlabs.com/learn/ghsa-864f-rcv7-6rh4-critical-type-confusion-vulnerability-in-isolated-vm)
- [DevOps.com — Critical Flaw in isolated-vm Can Lead to Sandbox Escape, RCE Threat](https://devops.com/critical-flaw-in-isolated-vm-can-lead-to-sandbox-escape-rce-threat/)

---

<div align="center">

**Status:** Patched and disclosed · **Advisory:** `GHSA-864f-rcv7-6rh4` · **CVE:** pending assignment

</div>
