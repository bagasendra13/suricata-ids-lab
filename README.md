# Lightweight Suricata IDS Lab

A lightweight Intrusion Detection System (IDS) laboratory built with
Suricata, Grafana Alloy, Grafana Loki, and Grafana.

This project demonstrates an end-to-end security monitoring pipeline,
from network traffic detection to centralized log collection and
visualization.

---

## Overview

The laboratory uses Suricata as the network intrusion detection engine.

Suricata inspects network traffic and applies custom detection rules.
Detected events are written to EVE JSON, collected by Grafana Alloy,
stored in Grafana Loki, and visualized through a Grafana IDS dashboard.

The project also includes a lightweight Nginx landing page that provides
a presentation-oriented entry point to the IDS dashboard.

---

## Architecture

```text
Network Traffic
      |
      v
   enp0s3
      |
      v
Suricata 7.0.3
      |
      v
Custom Detection Rules
      |
      v
EVE JSON
      |
      v
Grafana Alloy 1.20.1
      |
      v
Grafana Loki 3.7.8
      |
      v
LogQL
      |
      v
Grafana IDS Dashboard


---

## Components

| Component     | Version              | Role                          |
| ------------- | -------------------- | ----------------------------- |
| Ubuntu Server | 24.04.5 LTS          | Operating system              |
| Suricata      | 7.0.3                | Network IDS                   |
| Grafana Alloy | 1.20.1               | Log collector                 |
| Grafana Loki  | 3.7.8                | Log storage and query backend |
| Grafana       | Current installation | Visualization                 |
| Nginx         | 1.24.0               | Landing page                  |

---

## Environment

|Configuration|Value|
|---|---|
|VM Name|IDS-Suricata|
|Hostname|ids-suricata|
|CPU|2 vCPU|
|RAM|2 GB|
|Disk|20 GB VDI|
|Network|NAT|
|Interface|enp0s3|
|Guest IP|10.0.2.15/24|

---

## Suricata

Suricata captures traffic from the `enp0s3` interface using AF_PACKET.

Configuration:

```text
/etc/suricata/suricata.yaml
```

Custom rules:

```text
/var/lib/suricata/rules/local.rules
```

EVE JSON:

```text
/var/log/suricata/eve.json
```

The Suricata service is managed through:

```text
suricata.service
```

---

## Detection Rules

The laboratory contains nine custom detection rules.

|SID|Signature|Detection|
|---|---|---|
|1000002|LAB ICMP GOOGLE DNS TEST|ICMP traffic to Google DNS|
|1000003|LAB HTTP TEST|Generic HTTP traffic|
|1000004|LAB TCP PORT 80 TEST|TCP port 80|
|1000005|LAB HTTP EXAMPLE.COM TEST|HTTP host example.com|
|1000006|LAB HTTP GET EXAMPLE.COM TEST|HTTP GET + example.com|
|1000007|LAB EXACT HTTP GET EXAMPLE.COM ROOT TEST|GET + host + root URI|
|1000008|LAB Suspicious HTTP User-Agent|Suspicious sqlmap User-Agent|
|1000009|LAB Path Traversal Attempt|Path traversal test|
|1000010|LAB ICMP Host Discovery Attempt|ICMP echo request|

Detailed rule documentation is available in:

```text
docs/detection-rules.md
```

---

## Detection Coverage

### TCP

- TCP port 80
    
- HTTP traffic
    

### HTTP

- Generic HTTP detection
    
- HTTP host detection
    
- HTTP GET detection
    
- HTTP GET + Host detection
    
- HTTP GET + Host + URI detection
    
- Suspicious User-Agent detection
    
- Path traversal test detection
    

### ICMP

- ICMP Google DNS test
    
- ICMP host discovery attempt
    

---

## Severity

The custom rules demonstrate three severity levels:

```text
Severity 1
LAB Path Traversal Attempt

Severity 2
LAB Suspicious HTTP User-Agent
LAB ICMP Host Discovery Attempt

Severity 3
LAB ICMP GOOGLE DNS TEST
LAB HTTP TEST
LAB TCP PORT 80 TEST
LAB HTTP EXAMPLE.COM TEST
LAB HTTP GET EXAMPLE.COM TEST
LAB EXACT HTTP GET EXAMPLE.COM ROOT TEST
```

Severity values are available in the Suricata EVE JSON events and are  
visualized in Grafana.

---

## Logging Pipeline

The complete logging pipeline is:

```text
Suricata
   |
   v
/var/log/suricata/eve.json
   |
   v
Grafana Alloy
   |
   v
Grafana Loki
   |
   v
Grafana
```

Alloy reads the Suricata EVE JSON log and forwards the events to Loki.

Grafana then queries the stored events using LogQL.

Main Loki selector:

```text
{job="suricata"}
```

Alert filter:

```text
{job="suricata"}
| json
| event_type="alert"
```

---

## Grafana Dashboard

The IDS dashboard is functionally complete and contains nine main panels:

1. Total Alerts
    
2. Top Alert Signatures
    
3. Alert Severity
    
4. Top Source IP
    
5. Top Destination IP
    
6. Alert Timeline
    
7. Recent Suricata Alerts
    
8. SID Distribution
    
9. Protocol Distribution
    

A separate Signature Distribution panel was not included because its  
visualization was too similar to Top Alert Signatures.

---

## Recent Suricata Alerts

The dashboard displays recent alerts using a compact format:

```text
10.0.2.15 → 172.66.147.243 | TCP | SID 1000009 |
LAB Path Traversal Attempt | severity=1
```

Displayed information includes:

- Source IP
    
- Destination IP
    
- Protocol
    
- SID
    
- Signature
    
- Severity
    

The dashboard presentation intentionally avoids duplicate timestamps  
and unnecessary log-level indicators.

---

## Example Alerts

### ICMP Detection

```text
10.0.2.15 → 8.8.8.8
ICMP
SID 1000002
LAB ICMP GOOGLE DNS TEST
severity=3
```

### ICMP Host Discovery

```text
10.0.2.15 → 8.8.8.8
ICMP
SID 1000010
LAB ICMP Host Discovery Attempt
severity=2
```

### Path Traversal

```text
10.0.2.15 → 172.66.147.243
TCP
SID 1000009
LAB Path Traversal Attempt
severity=1
```

---

## Known Limitation

### SID 1000003 - Generic HTTP Detection

SID `1000003` intentionally uses a broad HTTP rule:

```text
alert http any any -> any any
```

Because the rule matches generic HTTP traffic, it can generate alert noise.

Normal browser traffic, application traffic, and Grafana-related HTTP  
traffic may trigger the rule.

This behavior is retained intentionally as a laboratory demonstration  
of the relationship between detection coverage and alert precision.

```text
Broad Rule
    |
    v
More Detection
    |
    v
More Alert Noise
    |
    v
Rule Refinement
```

The rule demonstrates why IDS rules often need refinement to reduce  
false positives and unnecessary alerts.

---

## Landing Page

The project includes an Nginx-based landing page.

The landing page provides:

- Project title
    
- System status
    
- Component versions
    
- IDS project information
    
- Direct navigation to the Grafana IDS dashboard
    

Conceptually:

```text
Browser
   |
   v
Nginx Landing Page
   |
   v
Open IDS Dashboard
   |
   v
Grafana IDS Dashboard
```

The landing page is a presentation layer and does not modify the  
underlying Suricata detection pipeline.

---

## Dashboard Access

The Grafana dashboard is available through the configured VirtualBox  
port forwarding.

The landing page provides a direct link to the dashboard so that users  
do not need to navigate through the default Grafana home page first.

---

## Project Structure

```text
suricata-ids-lab/
|
├── README.md
├── LICENSE
|
├── rules/
|   └── local.rules
|
├── config/
|   ├── alloy/
|   |   └── config.alloy
|   └── loki/
|       └── config.yaml
|
├── docs/
|   ├── architecture.md
|   ├── dashboard.md
|   ├── detection-rules.md
|   └── testing.md
|
└── screenshots/
```

---

## Security Notes

This repository is intended for laboratory and educational purposes.

Do not publish:

- Passwords
    
- Credentials
    
- API tokens
    
- Private keys
    
- SSH keys
    
- Secrets
    
- Raw Suricata logs
    
- Sensitive network information
    

Configuration files must be reviewed and sanitized before being  
committed to a public repository.

The `eve.json` and other raw log files should not be uploaded to the  
repository.

---

## Project Status

```text
Infrastructure        COMPLETE
Suricata              COMPLETE
Custom Detection      COMPLETE
EVE JSON              COMPLETE
Grafana Alloy         COMPLETE
Grafana Loki          COMPLETE
Grafana               COMPLETE
IDS Dashboard         COMPLETE
Landing Page          COMPLETE
Documentation         IN PROGRESS
Final Validation      PENDING
```

---

## Purpose

This project was created as a lightweight IDS laboratory to demonstrate:

- Network intrusion detection
    
- Custom Suricata rules
    
- Structured security logging
    
- Log collection
    
- Centralized log storage
    
- LogQL querying
    
- Security dashboard visualization
    
- Alert severity analysis
    
- IDS rule refinement
    

---

## Conclusion

The Lightweight Suricata IDS Lab demonstrates a complete security  
monitoring pipeline using open-source IDS and observability components.

Suricata performs network traffic inspection and detection. Grafana  
Alloy collects the resulting security events, Loki stores and queries  
the events, and Grafana visualizes the information through an IDS  
dashboard.

The project also demonstrates an important IDS principle: broader  
detection rules can increase visibility but may also generate additional  
alert noise, making rule refinement an important part of IDS development.
