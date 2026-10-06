````markdown
# Suricata Detection Rules

## Overview

This document describes the custom Suricata detection rules used in the
Lightweight Suricata IDS Lab.

The rules are designed for laboratory testing and demonstrate different
levels of detection precision across ICMP, TCP, and HTTP traffic.

Custom rules are stored in:

```text
/var/lib/suricata/rules/local.rules
```


---

## Rule Summary

|SID|Signature|Protocol|Severity|
|---|---|---|---|
|1000002|LAB ICMP GOOGLE DNS TEST|ICMP|3|
|1000003|LAB HTTP TEST|HTTP|3|
|1000004|LAB TCP PORT 80 TEST|TCP|3|
|1000005|LAB HTTP EXAMPLE.COM TEST|HTTP|3|
|1000006|LAB HTTP GET EXAMPLE.COM TEST|HTTP|3|
|1000007|LAB EXACT HTTP GET EXAMPLE.COM ROOT TEST|HTTP|3|
|1000008|LAB Suspicious HTTP User-Agent|HTTP|2|
|1000009|LAB Path Traversal Attempt|HTTP|1|
|1000010|LAB ICMP Host Discovery Attempt|ICMP|2|

---

## SID 1000002

### LAB ICMP GOOGLE DNS TEST

Purpose:

Detect ICMP traffic from the laboratory host to Google DNS.

Example traffic:

```text
10.0.2.15 → 8.8.8.8
```

Protocol:

```text
ICMP
```

Severity:

```text
3
```

The rule was tested successfully and the resulting event was observed  
in EVE JSON and Grafana.

---

## SID 1000003

### LAB HTTP TEST

Purpose:

Demonstrate broad HTTP detection.

The rule uses a generic HTTP condition:

```text
alert http any any -> any any
```

Severity:

```text
3
```

This rule intentionally has broad coverage.

### Known Behavior

Because the rule matches generic HTTP traffic, it can generate alerts  
from:

- Browser traffic
    
- Grafana traffic
    
- Application traffic
    
- Other HTTP requests
    

Therefore, this rule can generate significant alert noise.

This behavior is intentional for the laboratory and demonstrates an  
important IDS concept:

```text
Broad Detection
      |
      v
Higher Visibility
      |
      v
More Alert Noise
      |
      v
Rule Refinement
```

The rule is retained as a demonstration of the trade-off between  
coverage and precision.

---

## SID 1000004

### LAB TCP PORT 80 TEST

Purpose:

Detect TCP traffic associated with port 80.

Protocol:

```text
TCP
```

Severity:

```text
3
```

The rule was successfully detected and observed in the Suricata EVE  
JSON output and Grafana dashboard.

---

## SID 1000005

### LAB HTTP EXAMPLE.COM TEST

Purpose:

Detect HTTP traffic associated with the `example.com` host.

The rule uses HTTP host inspection and content matching.

Detection concept:

```text
HTTP
+
Host: example.com
```

Severity:

```text
3
```

The rule was tested and its host matching behavior was verified.

---

## SID 1000006

### LAB HTTP GET EXAMPLE.COM TEST

Purpose:

Detect an HTTP GET request associated with `example.com`.

Detection concept:

```text
HTTP
+
GET
+
Host: example.com
```

Severity:

```text
3
```

The rule was successfully detected in the laboratory environment.

---

## SID 1000007

### LAB EXACT HTTP GET EXAMPLE.COM ROOT TEST

Purpose:

Provide a more precise HTTP detection rule.

Detection concept:

```text
HTTP
+
GET
+
Host: example.com
+
URI: /
```

Severity:

```text
3
```

This rule demonstrates how multiple HTTP conditions can be combined to  
reduce the scope of a detection.

The rule was successfully tested and its more precise matching behavior  
was verified.

---

## SID 1000008

### LAB Suspicious HTTP User-Agent

Purpose:

Demonstrate detection based on a suspicious HTTP User-Agent.

The current rule detects:

```text
sqlmap
```

using HTTP User-Agent inspection.

Rule concept:

```text
alert http any any -> any any
(msg:"LAB Suspicious HTTP User-Agent";
 http.user_agent;
 content:"sqlmap";
 nocase;
 classtype:attempted-recon;
 sid:1000008;
 rev:1;)
```

Severity:

```text
2
```

The rule was successfully detected and verified in:

- Suricata alert output
    
- EVE JSON
    
- Grafana dashboard
    

---

## SID 1000009

### LAB Path Traversal Attempt

Purpose:

Demonstrate HTTP URI-based detection.

The laboratory test uses the marker:

```text
LAB-TRAVERSAL-TEST
```

Detection concept:

```text
HTTP URI
+
LAB-TRAVERSAL-TEST
```

Severity:

```text
1
```

The rule was successfully detected and verified in:

- Suricata alert output
    
- EVE JSON
    
- Grafana dashboard
    

This rule represents the highest severity level used in the current  
laboratory.

---

## SID 1000010

### LAB ICMP Host Discovery Attempt

Purpose:

Demonstrate detection of ICMP echo requests.

The rule uses:

```text
itype:8
```

which corresponds to an ICMP Echo Request.

Rule concept:

```text
alert icmp any any -> any any
(msg:"LAB ICMP Host Discovery Attempt";
 itype:8;
 classtype:attempted-recon;
 sid:1000010;
 rev:1;)
```

Severity:

```text
2
```

The rule was successfully tested and detected in the laboratory.

---

## Severity Classification

The current laboratory uses three severity levels.

### Severity 1

```text
LAB Path Traversal Attempt
```

### Severity 2

```text
LAB Suspicious HTTP User-Agent
LAB ICMP Host Discovery Attempt
```

### Severity 3

```text
LAB ICMP GOOGLE DNS TEST
LAB HTTP TEST
LAB TCP PORT 80 TEST
LAB HTTP EXAMPLE.COM TEST
LAB HTTP GET EXAMPLE.COM TEST
LAB EXACT HTTP GET EXAMPLE.COM ROOT TEST
```

The severity values are preserved in the Suricata EVE JSON events and  
are used by Grafana for visualization.

---

## Detection Overlap

A single network request can trigger multiple rules.

For example, an HTTP request to `example.com` may satisfy:

```text
LAB HTTP TEST
LAB TCP PORT 80 TEST
LAB HTTP EXAMPLE.COM TEST
LAB HTTP GET EXAMPLE.COM TEST
LAB EXACT HTTP GET EXAMPLE.COM ROOT TEST
```

This is expected behavior.

Each rule evaluates the traffic independently.

Therefore:

```text
1 Network Request
       |
       +---- Rule A
       |
       +---- Rule B
       |
       +---- Rule C
       |
       +---- Rule D
       |
       +---- Rule E
```

This explains why the dashboard can display multiple alerts generated  
from a single network activity.

---

## Rule Validation

The rules were validated through the complete detection pipeline:

```text
Traffic
   |
   v
Suricata
   |
   v
Alert
   |
   v
EVE JSON
   |
   v
Grafana Alloy
   |
   v
Grafana Loki
   |
   v
Grafana Dashboard
```

Validation confirmed that the custom detection events are visible in  
the final dashboard.

---

## Legacy Rule

An older rule using SID:

```text
100001
```

is not part of the current detection set.

The active laboratory rules use:

```text
1000002
1000003
1000004
1000005
1000006
1000007
1000008
1000009
1000010
```

---

## Laboratory Scope

These rules are designed specifically for controlled laboratory  
demonstration.

They should not be interpreted as a complete production IDS rule set.

Production deployments would normally require:

- Broader protocol coverage
    
- Rule tuning
    
- False-positive analysis
    
- Thresholding
    
- Context-aware detection
    
- Network-specific tuning
    
- Signature maintenance
    
- Regular rule updates
    

---

## Conclusion

The custom rule set demonstrates multiple IDS detection strategies,  
including protocol-based detection, port-based detection, HTTP host  
matching, HTTP method matching, URI matching, User-Agent inspection,  
and ICMP type detection.

The laboratory also demonstrates the importance of balancing detection  
coverage with alert precision.

The broad HTTP rule intentionally generates additional noise, while  
more specific rules demonstrate how detection conditions can be refined.
````