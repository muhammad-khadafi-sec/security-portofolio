# Threat Model

## Scope

This threat model focuses on the primary security risks associated
with identity-based access control, segmented data center resources,
and centralized security monitoring.

## Assets

- Sensitive data center resources
- User identities
- Privileged accounts
- Authentication services
- Access-control policies
- Security telemetry
- SOC infrastructure

## Trust Boundaries

### User / Access Request

All access requests are treated as untrusted until evaluated by
the applicable access-control policies.

### Data Center Zones

Each security zone represents a separate security boundary and
requires appropriate authorization before access is granted.

### Management Zone

The management environment is isolated from normal production
access paths and is intended for controlled administrative access.

## Threat Scenarios

### Compromised User Identity

An attacker obtains valid user credentials and attempts to access
protected resources.

**Potential impact:**
- Unauthorized access
- Lateral movement
- Data exposure

**Mitigations:**
- MFA
- RBAC
- Least privilege
- Policy-based access control
- Continuous monitoring

### Privileged Account Compromise

An attacker compromises a privileged account.

**Potential impact:**
- Administrative access
- Security-control modification
- Broad system compromise

**Mitigations:**
- PAM
- MFA
- Privileged session controls
- Administrative access logging

### Lateral Movement

An attacker attempts to move between security zones after gaining
initial access.

**Potential impact:**
- Expansion of compromise
- Access to sensitive systems

**Mitigations:**
- Network segmentation
- Micro-segmentation
- Resource-level authorization
- Continuous monitoring

### Policy Misconfiguration

An incorrect access-control policy unintentionally grants excessive
permissions.

**Potential impact:**
- Unauthorized access
- Excessive privileges

**Mitigations:**
- Policy review
- Change control
- Access testing
- Continuous monitoring

### Compromised Endpoint

A legitimate endpoint becomes compromised after access has already
been granted.

**Potential impact:**
- Credential theft
- Lateral movement
- Malicious activity from an authorized device

**Mitigations:**
- Endpoint detection and response
- Security telemetry
- Risk-based policy evaluation
- Access re-evaluation