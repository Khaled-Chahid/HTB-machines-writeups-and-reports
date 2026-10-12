## Assessment Summary

An unpatched implementation of the SMBv1 protocol (MS17-010 / EternalBlue) allowed unauthenticated remote code execution. A crafted exploit triggered a reverse shell that ran as `NT AUTHORITY\SYSTEM`, giving full control of the Windows host.

**Scope:** 10.10.10.40 in the HTB lab. This report is based on the supplied official writeup; the findings were not independently retested.

| ID | Finding | Severity |
|---|---|---|
| BLU-01 | Unpatched SMBv1 (MS17-010 / EternalBlue) RCE | Critical |
| BLU-02 | Anonymous/Guest SMB access enabled | Medium |

---

### BLU-01 — Unpatched SMBv1 (MS17-010 / EternalBlue) Remote Code Execution

**Affected service:** SMB (ports 139, 445), Windows 7.

The host runs a version of SMBv1 vulnerable to MS17-010. A specially crafted SMB packet allows an attacker to execute arbitrary code on the target without valid credentials, by exploiting a flaw in Microsoft's handling of crafted SMB requests.

**Verification:** An nmap vulnerability scan (`--script vuln`) flagged the host as vulnerable to EternalBlue. A public, non-Metasploit exploit (Exploit-DB #42315) was adapted with a custom MSFvenom reverse-shell payload and executed against the host, returning a shell running as `NT AUTHORITY\SYSTEM`.

**Impact:** Complete compromise of the host by any attacker with network access to SMB, requiring no prior credentials. This is the most severe class of finding, as it combines unauthenticated access with full SYSTEM-level code execution.

**Remediation:** Apply the MS17-010 security update immediately. Disable SMBv1 entirely in favor of SMBv2/v3. Restrict SMB exposure to trusted internal networks and block ports 139/445 at the perimeter.

---

### BLU-02 — Anonymous/Guest SMB Access Enabled

**Affected service:** SMB (ports 139, 445).

The host permits guest-level authentication to SMB, allowing unauthenticated enumeration of shares, users, and system details without a valid account.

**Verification:** `enum4linux -a 10.10.10.40` confirmed that guest login was supported, enabling full enumeration of the host.

**Impact:** Attackers can gather detailed reconnaissance (usernames, shares, OS/service details) without credentials, materially lowering the effort needed to plan further attacks such as BLU-01.

**Remediation:** Disable the Guest account and reject anonymous/null SMB sessions. Require authentication for all SMB access and audit share permissions to enforce least privilege.

---

### Remediation Priority

First apply the MS17-010 patch and disable SMBv1, since this is the critical unauthenticated path to full compromise. Then disable guest/anonymous SMB access to close the reconnaissance vector. Verify patch status and SMB configuration with a follow-up vulnerability scan.