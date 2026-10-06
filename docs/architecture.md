````markdown
# System Architecture

## Overview

The Lightweight Suricata IDS Lab is designed as a lightweight,
end-to-end network intrusion detection and security monitoring pipeline.

The architecture connects Suricata, Grafana Alloy, Grafana Loki,
and Grafana into a single monitoring workflow.

---

## Architecture Diagram

```text
                    Network Traffic
                          |
                          v
                    enp0s3 Interface
                          |
                          v
                  +------------------+
                  |    Suricata      |
                  |     7.0.3        |
                  |   Network IDS     |
                  +--------+---------+
                           |
                           | Detection Events
                           v
                  +------------------+
                  |     EVE JSON     |
                  |   eve.json       |
                  +--------+---------+
                           |
                           v
                  +------------------+
                  |  Grafana Alloy   |
                  |     1.20.1       |
                  |   Log Collector  |
                  +--------+---------+
                           |
                           | Log forwarding
                           v
                  +------------------+
                  |   Grafana Loki   |
                  |      3.7.8       |
                  |   Log Storage    |
                  +--------+---------+
                           |
                           | LogQL
                           v
                  +------------------+
                  |     Grafana      |
                  |  IDS Dashboard   |
                  +------------------+


---

## 1. Network Layer

The laboratory uses a single Ubuntu Server virtual machine.

Network configuration:

|Property|Value|
|---|---|
|Interface|`enp0s3`|
|IP Address|`10.0.2.15/24`|
|Network Mode|NAT|
|Capture Method|AF_PACKET|

Suricata monitors network traffic received through the `enp0s3`  
interface.

---

## 2. Suricata IDS

Suricata is the primary intrusion detection engine.

Version:

```text
7.0.3
```

Configuration:

```text
/etc/suricata/suricata.yaml
```

Custom rules:

```text
/var/lib/suricata/rules/local.rules
```

Service:

```text
suricata.service
```

Suricata analyzes packets and applies the configured detection rules.

When a rule matches network traffic, Suricata generates an alert event.

---

## 3. EVE JSON

Suricata writes structured security events to:

```text
/var/log/suricata/eve.json
```

The EVE JSON format provides structured fields such as:

- Event type
    
- Timestamp
    
- Source IP
    
- Destination IP
    
- Protocol
    
- Alert signature ID
    
- Alert signature
    
- Alert severity
    

This structured format makes the events suitable for log collection  
and further analysis.

---

## 4. Grafana Alloy

Grafana Alloy acts as the log collection layer.

Version:

```text
1.20.1
```

Configuration:

```text
/etc/alloy/config.alloy
```

Service:

```text
alloy.service
```

Alloy reads the Suricata EVE JSON log and forwards the events to  
Grafana Loki.

The data flow is:

```text
eve.json
   |
   v
Grafana Alloy
   |
   v
Grafana Loki
```

---

## 5. Grafana Loki

Grafana Loki provides centralized log storage and querying.

Version:

```text
3.7.8
```

Configuration:

```text
/etc/loki/config.yaml
```

Service:

```text
loki.service
```

Port:

```text
3100
```

Loki stores the Suricata events received from Alloy.

The main LogQL selector used by the dashboard is:

```text
{job="suricata"}
```

Alert events can be filtered using:

```text
{job="suricata"}
| json
| event_type="alert"
```

---

## 6. Grafana

Grafana provides the visualization layer.

The Grafana dashboard queries Loki and presents Suricata security  
events in a human-readable format.

The dashboard includes:

- Total Alerts
    
- Top Alert Signatures
    
- Alert Severity
    
- Top Source IP
    
- Top Destination IP
    
- Alert Timeline
    
- Recent Suricata Alerts
    
- SID Distribution
    
- Protocol Distribution
    

The Signature Distribution panel was intentionally omitted because its  
visualization was too similar to Top Alert Signatures.

---

## 7. Nginx Landing Page

Nginx provides a lightweight presentation layer for the project.

The landing page is served through:

```text
HTTP port 80
```

The landing page provides:

- Project title
    
- IDS system status
    
- Component information
    
- Link to the Grafana IDS dashboard
    

The landing page does not replace Grafana. It acts as the entry point  
for the project presentation.

The logical flow is:

```text
Browser
   |
   v
Nginx Landing Page
   |
   v
Grafana IDS Dashboard
```

---

## 8. Complete Data Flow

The complete end-to-end pipeline is:

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
Detection Event
      |
      v
/var/log/suricata/eve.json
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
```

---

## 9. Detection and Visualization Relationship

A detected network event follows this process:

```text
Traffic
   |
   v
Rule Matching
   |
   v
Suricata Alert
   |
   v
EVE JSON
   |
   v
Alloy
   |
   v
Loki
   |
   v
LogQL Query
   |
   v
Grafana Panel
```

This allows a single detection event to become visible through several  
dashboard views.

For example, an alert can contribute to:

- Total alert count
    
- Severity distribution
    
- Source IP statistics
    
- Destination IP statistics
    
- Protocol distribution
    
- SID distribution
    
- Alert timeline
    
- Recent alert list
    

---

## 10. Severity Flow

Suricata alert severity is preserved throughout the pipeline.

```text
Suricata Rule
      |
      v
Alert Severity
      |
      v
EVE JSON
      |
      v
Alloy
      |
      v
Loki
      |
      v
Grafana
```

The current laboratory demonstrates severity levels:

```text
1
2
3
```

This allows the dashboard to provide a basic representation of alert  
priority.

---

## 11. Design Goals

The architecture was designed around several goals:

### Lightweight

The entire monitoring stack runs inside a small virtual machine with:

- 2 vCPU
    
- 2 GB RAM
    
- 20 GB virtual disk
    

### Modular

Each component has a specific responsibility:

```text
Suricata  → Detection
EVE JSON  → Structured Events
Alloy     → Collection
Loki      → Storage
Grafana   → Visualization
Nginx     → Presentation
```

### Observable

The pipeline allows detected events to be followed from network traffic  
through to the final dashboard.

### Educational

The architecture demonstrates fundamental IDS concepts without requiring  
a large distributed infrastructure.

---

## 12. Security Considerations

The project is intended for a controlled laboratory environment.

The following information should not be exposed in a public repository:

- Passwords
    
- Credentials
    
- API tokens
    
- Private keys
    
- SSH keys
    
- Secrets
    
- Raw security logs
    
- Sensitive network information
    

Configuration files should be reviewed and sanitized before publication.

---

## Conclusion

The architecture provides a complete lightweight security monitoring  
pipeline.

Suricata performs network detection, EVE JSON provides structured event  
data, Alloy collects the events, Loki stores and queries them, and  
Grafana visualizes the resulting security information.

Nginx provides a simple presentation layer that connects the project  
landing page to the IDS dashboard.
````