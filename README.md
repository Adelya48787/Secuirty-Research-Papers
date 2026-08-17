<div align="center">

# CVE-2026-25769

**Remote Code Execution on the Wazuh Master Node through a Compromised Worker Node**

[![CVE](https://img.shields.io/badge/CVE-2026--25769-red?style=for-the-badge)](https://nvd.nist.gov/vuln/detail/CVE-2026-25769)
[![Severity](https://img.shields.io/badge/Severity-CRITICAL-critical?style=for-the-badge)](#)
[![CVSS](https://img.shields.io/badge/CVSS%203.1-9.1-orange?style=for-the-badge)](#)
[![CWE](https://img.shields.io/badge/CWE-502%20Deserialization-blue?style=for-the-badge)](https://cwe.mitre.org/data/definitions/502.html)
[![Status](https://img.shields.io/badge/Status-Patched-brightgreen?style=for-the-badge)](#)

</div>

---

## Overview

**Wazuh** is an open-source platform for threat prevention, detection, and response, commonly deployed in a **master/worker cluster architecture**. Versions **4.0.0 through 4.14.2** contain a critical **Remote Code Execution (RCE)** vulnerability caused by **unsafe deserialization of untrusted data** in the cluster communication layer.

Any organization where a worker node is compromised — through initial access, an insider threat, or a supply-chain attack — can be pivoted into **full root-level RCE on the master node**, which aggregates every alert, log, and detection rule across the deployment.

> ⚠️ This document summarizes publicly disclosed vulnerability details for defensive and educational purposes. It does not contain a working exploit.

---

## At a Glance

| Field | Value |
|---|---|
| **CVE ID** | [CVE-2026-25769](https://nvd.nist.gov/vuln/detail/CVE-2026-25769) |
| **Severity** | Critical |
| **CVSS 3.1 Score** | `9.1` (`AV:N/AC:L/PR:H/UI:N/S:C/C:H/I:H/A:H`) |
| **CWE** | [CWE-502 — Deserialization of Untrusted Data](https://cwe.mitre.org/data/definitions/502.html) |
| **Affected Versions** | `4.0.0` – `4.14.2` |
| **Patched Version** | `4.14.3` |
| **Attack Vector** | Network (TCP/1516, cluster channel) |
| **Privileges Required** | High (authenticated worker node) |
| **Impact** | Full RCE as root on master node |

---

## Architecture Context

In a Wazuh cluster, one **master** coordinates one or more **worker nodes**, all communicating over **TCP port 1516**. Messages are encrypted with a shared **Fernet key** defined in `ossec.conf`, ensuring only nodes possessing the correct key can exchange data.

The flaw is not in the encryption — it's in what happens **after** decryption: **the master implicitly trusts the content of any message from an authenticated worker.**

```
 🔒
 Master Server ───── TCP/1516 (Fernet-encrypted) ─────▶ Worker Node ✅
 │
 ├──── TCP/1516 ─────▶ Worker Node ✅
 │
 └──── TCP/1516 ─────▶ Worker Node ✅ (compromised)
```

Once a worker is compromised, the attacker already possesses everything needed to send **authenticated, trusted, malicious messages** to the master.

---


## ⛓️ Attack Chain

```mermaid
sequenceDiagram
    participant A as Attacker
    participant W as Compromised Worker
    participant M as Master Node (root)

    A->>W: 1. Gain access (initial access / insider / supply chain)
    Note over W: Worker already holds the shared Fernet key
    A->>W: 2. Craft malicious DAPI request via LocalClient.execute()
    W->>M: 3. Send Fernet-encrypted message over TCP/1516
    M->>M: 4. APIRequestQueue.run() decrypts & calls json.loads()
    M->>M: 5. as_wazuh_object() imports subprocess, returns getoutput
    M->>M: 6. DistributedAPI.run_local() executes subprocess.getoutput(cmd=...)
    M-->>A: 7. Arbitrary command output — RCE as root
```

---


### Primary Fix
**Upgrade to Wazuh `4.14.3` or later.** The patch introduces an allowlist that blocks arbitrary module imports during deserialization.



<div align="center">

*This REPORT is intended for defensive security research, patch prioritization, and awareness. Always test upgrades in a staging environment before applying to production clusters.*

</div>
