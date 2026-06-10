# VPC Flow Log Analysis Report

Account: 8455xxxxxxxx
Region: us-east-1
Log Group: /aws/vpc/flowlogs
Date: 2026-06-10

---

## Traffic Summary

- Total records scanned: 305
- Accepted connections observed: 155
- Rejected SSH attempts detected: 3
- Port scan activity detected: None

---

## Top Destinations

Query: Top destination IPs by total bytes (ACCEPT traffic)

1. 71,334,113 bytes (~71 MB) - primary destination IP (AWS internal traffic)

---

## Security Findings

1. Rejected SSH Attempts Detected

Two external sources attempted SSH connections (port 22) that were rejected by the security group:

- Source: 145.90.65.82 - 2 rejected attempts
- Source: 205.210.31.53 - 1 rejected attempt

These IPs are external and were not granted SSH access. The security group is functioning correctly by blocking unauthorized inbound SSH.

2. Port Scan Activity

No sources were detected attempting connections across more than 20 unique ports. No port scanning activity identified in the observation window.

---

## Remediation Actions

- Rejected SSH sources (145.90.65.82, 205.210.31.53) noted for monitoring
- CloudWatch metric filter and alarm configured to alert on rejected SSH attempts exceeding threshold of 10 per 5-minute window
- No NACL changes required at this time given low volume

---

## Recommendations

- Enable GuardDuty for automated threat detection and IP reputation scoring
- Restrict SSH access in the security group to known IP ranges rather than 0.0.0.0/0
- Review and rotate EC2 key pairs quarterly
- Set up VPC Flow Log retention policy to manage CloudWatch Logs storage costs

---
