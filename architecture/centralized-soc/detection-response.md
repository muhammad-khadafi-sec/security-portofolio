# Detection and Response Workflow

## Scenario

A potentially compromised endpoint is detected at a remote site.

## Workflow

```text
Remote Endpoint
      |
      v
EDR Detection
      |
      v
Log Forwarder
      |
      v
Encrypted VPN
      |
      v
Centralized SIEM
      |
      v
Correlation & Detection
      |
      v
Context Enrichment
      |
      v
SOAR
      |
      +----> Incident Ticket
      |
      +----> Malware Sandbox
      |
      +----> Network Enforcement
      |
      +----> Endpoint Response

Detection

The endpoint generates a security event which is forwarded to the
centralized SOC.

Correlation

The SIEM correlates the endpoint event with available network,
asset, vulnerability, and threat intelligence context.

Investigation

The SOC analyst reviews the enriched event and determines whether
the activity represents a confirmed security incident.

Response

Depending on the incident severity, SOAR may initiate response
actions such as:

Creating an incident ticket
Submitting artifacts for malware analysis
Applying network enforcement
Initiating endpoint response

High-impact automated actions should include appropriate safeguards
to reduce the risk of false-positive driven disruption.
