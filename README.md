# Lab M8.04 - Enable Flow Logs + Interpret Traffic Patterns

**Repository:** [https://github.com/cloud-engineering-bootcamp/ce-lab-enable-flow-logs-interpret-traffic](https://github.com/cloud-engineering-bootcamp/ce-lab-enable-flow-logs-interpret-traffic)

**Activity Type:** Individual  
**Estimated Time:** 60 minutes

## Learning Objectives

- [ ] Enable VPC Flow Logs for network monitoring
- [ ] Configure CloudWatch Logs for flow log storage
- [ ] Query flow logs using CloudWatch Insights
- [ ] Detect security threats using flow log analysis

## Prerequisites

- [ ] VPC with EC2 instances and security groups
- [ ] Completed Module 8 Lesson 4

## Task

Enable VPC Flow Logs, generate traffic, analyze logs to detect accepted/rejected connections, and create security alerts.

## Step-by-Step Instructions

### Step 1: Enable VPC Flow Logs

```hcl
# Terraform
resource "aws_flow_log" "main" {
  vpc_id          = aws_vpc.main.id
  traffic_type    = "ALL"
  iam_role_arn    = aws_iam_role.flow_logs.arn
  log_destination = aws_cloudwatch_log_group.flow_logs.arn
}

resource "aws_cloudwatch_log_group" "flow_logs" {
  name              = "/aws/vpc/flowlogs"
  retention_in_days = 7
}
```

Or via CLI:

```bash
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-abc123 \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs \
  --deliver-logs-permission-arn arn:aws:iam::123456789012:role/flowlogsRole
```

### Step 2: Generate Test Traffic

```bash
# SSH to your EC2 instance (should succeed)
ssh ec2-user@<your-instance-ip>

# Try SSH to a blocked port (should fail)
telnet <your-instance-ip> 23
```

### Step 3: Query Flow Logs

**Query 1: Top 10 destination IPs**
```sql
fields @timestamp, dstAddr, sum(bytes) as totalBytes
| filter action = "ACCEPT"
| stats sum(bytes) as totalBytes by dstAddr
| sort totalBytes desc
| limit 10
```

**Query 2: Rejected SSH attempts**
```sql
fields @timestamp, srcAddr, dstPort
| filter dstPort = 22 and action = "REJECT"
| stats count() as attempts by srcAddr
| sort attempts desc
```

**Query 3: Port scanning detection**
```sql
fields @timestamp, srcAddr, dstPort
| filter action = "REJECT"
| stats count_distinct(dstPort) as uniquePorts by srcAddr
| filter uniquePorts > 20
| sort uniquePorts desc
```

### Step 4: Create CloudWatch Alarm

```bash
# Alarm for excessive rejected SSH attempts
aws cloudwatch put-metric-alarm \
  --alarm-name rejected-ssh-attempts \
  --metric-name RejectedSSHAttempts \
  --namespace CustomMetrics \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 100 \
  --comparison-operator GreaterThanThreshold \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:security-alerts
```

### Step 5: Document Findings

Create `flow-log-analysis.md`:

```markdown
# VPC Flow Log Analysis Report

## Traffic Summary
- Total connections: 1,523
- Accepted: 1,489 (98%)
- Rejected: 34 (2%)

## Top Destinations
1. 52.216.1.1 (AWS S3) - 245 MB
2. 54.239.2.1 (AWS API) - 89 MB
3. 203.0.113.42 (External API) - 12 MB

## Security Findings
1. **Port Scan Detected**
   - Source: 198.51.100.5
   - Unique ports attempted: 35
   - Action: Added to NACL deny list

2. **SSH Brute Force Attempt**
   - Source: 192.0.2.10
   - Rejected attempts: 127
   - Action: Blocked via WAF

## Recommendations
- Enable GuardDuty for automated threat detection
- Implement rate limiting at ALB
- Review security group rules quarterly
```

## Submission

- `flow-log-analysis.md` with findings
- Screenshots of CloudWatch Insights queries
- CloudWatch Alarm configuration

## Verification Checklist

- [ ] VPC Flow Logs enabled and logging
- [ ] Ran at least 3 CloudWatch Insights queries
- [ ] Identified at least 1 security concern
- [ ] Created CloudWatch Alarm

**Good luck! 🔒**
