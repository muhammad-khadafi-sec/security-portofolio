# Threat Model

## Scope

This threat model focuses on the security boundaries and major
failure risks within the proposed multi-site SOC architecture.

## Assets

- Security telemetry
- Centralized log storage
- SIEM
- SOAR
- Threat intelligence data
- Asset inventory
- Remote security sensors
- VPN transport

## Trust Boundaries

### Remote Sites

Remote sites are considered low-trust environments because
sensors or endpoints may potentially be compromised.

### Internet

The Internet is considered an untrusted network.

### Central SOC

The centralized SOC is considered a trusted security zone,
subject to internal access controls and hardening.

## Threat Scenarios

### Compromised Remote Sensor

An attacker compromises an IDS or EDR sensor and attempts to
inject, modify, or suppress security telemetry.

**Potential impact:**
- False telemetry
- Missed detections
- Incorrect incident context

**Mitigations:**
- Sensor hardening
- Secure communication
- Integrity monitoring
- Centralized validation

### VPN Compromise or Misconfiguration

Improper VPN configuration could expose or intercept telemetry.

**Potential impact:**
- Telemetry disclosure
- Telemetry manipulation
- Loss of monitoring visibility

**Mitigations:**
- Strong cryptographic configuration
- Restricted VPN access
- Network segmentation
- Continuous monitoring

### Log Collector Failure

A centralized collector becomes unavailable or overloaded.

**Potential impact:**
- Telemetry loss
- Delayed detection
- Monitoring blind spots

**Mitigations:**
- Capacity planning
- Redundancy
- Health monitoring
- Scalable ingestion architecture

### SOAR Automation Abuse

An automated response playbook executes an inappropriate action
because of a false positive or maliciously manipulated alert.

**Potential impact:**
- Unnecessary endpoint isolation
- Network disruption
- Operational impact

**Mitigations:**
- Approval gates for high-impact actions
- Playbook validation
- Alert confidence scoring
- Audit logging