# Assessment Summary

Weak credentials on the Tomcat Manager allowed authenticated application deployment. A deployed WAR payload executed as `NT AUTHORITY\SYSTEM`, giving full control of the Windows host.

Scope: `10.129.136.9:8080` in the HTB lab. This report is based on the supplied official writeup; the findings were not independently retested.

| ID | Finding | Severity |
|---|---|---|
| JER-01 | Weak credentials on an exposed Tomcat Manager | High |
| JER-02 | Tomcat application execution with SYSTEM privileges | High |

## JER-01 — Weak Tomcat Manager Credentials

**Affected service:** Tomcat Manager at `/manager/html`.

The management interface accepted the common credential pair `tomcat:s3cret`. The account had permission to deploy WAR applications, allowing an authenticated attacker to execute code on the server.

**Verification:** The `auxiliary/scanner/http/tomcat_mgr_login` module identified the credentials. The `exploit/multi/http/tomcat_mgr_upload` module then deployed a WAR payload and established a remote session.

**Impact:** Remote code execution by anyone able to reach the Manager interface and discover the weak credentials. WAR deployment is an administrative feature; the weakness is the inadequate protection of an account authorized to use it.

**Remediation:** Replace the credentials with a unique, strong password. Restrict Manager access to trusted administration hosts or a VPN, remove unused management accounts, and disable the interface when it is not needed.

## JER-02 — Tomcat Executes Applications as SYSTEM

**Affected component:** Tomcat service execution context.

The deployed application obtained a Windows shell running as `NT AUTHORITY\SYSTEM`. Application-level code execution therefore resulted directly in host-level administrative access.

**Verification:** After entering the command shell from Meterpreter, check the process identity:

```cmd
whoami
```

The official writeup documents SYSTEM access and access to the flag file under the Administrator profile.

**Impact:** An attacker who achieves code execution in Tomcat can read or modify protected files, change system settings, and establish persistence with SYSTEM privileges. This finding requires a code-execution path such as JER-01; it is not a separate unauthenticated entry point.

**Remediation:** Run Tomcat under a dedicated service account with only the permissions it needs. Remove local administrator privileges and restrict write access to application and configuration directories.

## Remediation Priority

First replace the Manager credentials and restrict access to the interface. Then reduce the Tomcat service account's privileges. Verify that deployment is limited to authorized administrators and that application processes no longer run as SYSTEM.


