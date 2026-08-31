# 🔓 SSE DNS Rebinding via CORS Misconfiguration

[![CVE](https://img.shields.io/badge/CVE-2026--9739-red?style=for-the-badge)](https://nvd.nist.gov/vuln/detail/cve-2026-9739)
[![Severity](https://img.shields.io/badge/Severity-High-orange?style=for-the-badge)]()
[![Type](https://img.shields.io/badge/Vulnerability-CORS%20Misconfiguration-blue?style=for-the-badge)]()
[![Status](https://img.shields.io/badge/Status-Patched-brightgreen?style=for-the-badge)]()

> A DNS rebinding attack made possible by an overly permissive `Access-Control-Allow-Origin: *` header on an SSE (Server-Sent Events) endpoint, allowing unauthorized origins to hijack sessions and execute tools on behalf of a victim.

---


This vulnerability enables **DNS rebinding attacks** against an SSE-based service. The root cause is a hardcoded, wildcard CORS header that fails to restrict which origins may open a persistent connection to the server. Under certain conditions, this allows a malicious website to trick a victim's browser into establishing an SSE connection with an internal/local service, leaking data that should never leave the trust boundary.

| Property | Detail |
|---|---|
| **Vulnerability Class** | CORS Misconfiguration → DNS Rebinding |
| **Root Cause** | Hardcoded `Access-Control-Allow-Origin: *` |
| **Affected Component** | `internal/server/mcp.go:370` — `sseHandler` |
| **Impact** | Session hijacking, unauthorized tool execution, internal data exposure |

---


## Attack Flow

1. Victim visits `attacker.com`, which initially resolves to the attacker's own server and loads malicious JavaScript.
2. The attacker lets the DNS TTL expire, then re-points `attacker.com` to `127.0.0.1` or an internal service IP.
3. The malicious script — still executing in the context of `attacker.com` — opens an SSE connection to the now-rebound address.
4. Because the server does not validate the `Host` or `Origin` header and echoes `Access-Control-Allow-Origin: *`, the connection is accepted.
5. The server streams events (including session-sensitive data) directly to the attacker-controlled script.



## Impact

- **Session hijacking** — SSE-delivered session identifiers can be captured by an unauthorized origin.
- **Unauthorized tool execution** — an attacker can trigger backend tools on behalf of the victim.
- **Internal network exposure** — internal-only services become reachable from the public web via the victim's browser.





<p align="center">
  <sub>Documented for educational and defensive security purposes.</sub>
</p>
