# Forecast vs Actual Cost Variance Report

**Date:** April 2, 2026
**Account:** Current AWS account
**Baseline (Estimate):** February 2026 actual spend (used as forecast for March)
**Comparison (Actual):** March 2026 actual spend
**Method:** Feb spend as proxy estimate → compare against Mar actuals

---

## Summary

| Metric | Value |
|--------|-------|
| Estimated (Feb 2026) | $6,479.21 |
| Actual (Mar 2026) | $7,394.01 |
| Total Variance | +$914.80 (+14.1%) |
| Estimate Accuracy | 87.6% |
| Status | REVIEW |
| Cost Anomalies Detected | 0 |

### AWS ML Forecast (Apr–Jun 2026)

| Month | Forecast (Mean) | Lower Bound | Upper Bound |
|-------|----------------|-------------|-------------|
| Apr 2026 | $6,701.27 | $6,547.07 | $6,855.47 |
| May 2026 | $6,899.17 | $6,624.61 | $7,173.72 |
| Jun 2026 | $6,677.19 | $6,327.75 | $7,026.63 |
| 3-Month Total | $20,277.63 | | |

AWS's ML forecast predicts a slight decrease from March levels, suggesting March may have been an above-average month.

---

## Service-Level Variance (Services > $5/month)

### INVESTIGATE (>25% variance) — 7 services

| Service | Feb Estimate | Mar Actual | Variance | % | Driver |
|---------|-------------|------------|----------|---|--------|
| Amazon Bedrock Service | $32.49 | $213.97 | +$181.49 | +558.6% | Significant increase in Bedrock model usage |
| Amazon Bedrock AgentCore | $127.35 | $637.09 | +$509.74 | +400.3% | Agent workload scaling — largest absolute variance |
| Kiro | $7.02 | $43.32 | +$36.30 | +516.8% | Increased Kiro usage |
| AWS CloudTrail | $114.73 | $234.39 | +$119.65 | +104.3% | Doubled — likely increased API activity or data events enabled |
| Amazon Redshift | $761.04 | $957.20 | +$196.16 | +25.8% | Cluster scaling or increased query volume |
| Amazon QuickSight | $61.21 | $78.90 | +$17.68 | +28.9% | Additional users or SPICE capacity |
| Amazon DocumentDB | $53.24 | $13.64 | -$39.60 | -74.4% | Significant reduction — instance downsized or stopped |
| Amazon Elastic Load Balancing | $45.46 | $24.50 | -$20.97 | -46.1% | Load balancers removed or consolidated |

### REVIEW (10–25% variance) — 7 services

| Service | Feb Estimate | Mar Actual | Variance | % |
|---------|-------------|------------|----------|---|
| Amazon VPC | $375.87 | $436.35 | +$60.48 | +16.1% |
| Amazon EC2 Compute | $835.81 | $705.10 | -$130.71 | -15.6% |
| EC2 - Other | $450.02 | $395.99 | -$54.03 | -12.0% |
| Amazon ECS | $194.74 | $221.52 | +$26.78 | +13.8% |
| Amazon Macie | $6.45 | $7.70 | +$1.24 | +19.3% |
| Amazon ElastiCache | $32.26 | $35.57 | +$3.31 | +10.3% |
| Amazon Kinesis | $42.60 | $38.19 | -$4.42 | -10.4% |

### OK (≤10% variance) — 17 services

| Service | Feb Estimate | Mar Actual | Variance | % |
|---------|-------------|------------|----------|---|
| Amazon OpenSearch Service | $1,143.37 | $1,162.65 | +$19.28 | +1.7% |
| AmazonCloudWatch | $508.47 | $520.22 | +$11.75 | +2.3% |
| Amazon EKS | $940.80 | $899.88 | -$40.92 | -4.3% |
| Amazon RDS | $260.10 | $253.99 | -$6.10 | -2.3% |
| Amazon GuardDuty | $167.51 | $171.80 | +$4.29 | +2.6% |
| Amazon SageMaker | $67.75 | $74.04 | +$6.29 | +9.3% |
| Amazon MemoryDB | $64.51 | $70.56 | +$6.05 | +9.4% |
| AWS Directory Service | $53.70 | $58.95 | +$5.25 | +9.8% |
| AWS KMS | $29.91 | $30.04 | +$0.13 | +0.4% |
| Amazon CodeWhisperer | $19.00 | $18.39 | -$0.61 | -3.2% |
| Amazon DynamoDB | $18.20 | $18.25 | +$0.05 | +0.3% |
| Amazon Managed Grafana | $18.00 | $18.00 | $0.00 | 0.0% |
| AWS Global Accelerator | $16.80 | $18.38 | +$1.57 | +9.4% |
| AWS WAF | $6.00 | $5.93 | -$0.07 | -1.2% |
| AWS Secrets Manager | $6.17 | $6.31 | +$0.15 | +2.4% |
| Amazon S3 | $5.54 | $5.83 | +$0.29 | +5.2% |
| Amazon Macie | $6.45 | $7.70 | +$1.24 | +19.3% |

---

## Root Cause Analysis

### Top 3 Cost Increases

1. **Bedrock AgentCore (+$509.74, +400%)** — The largest absolute increase. Agent workloads scaled significantly from Feb to Mar. This suggests new agent deployments or increased agent invocation volume. Recommend monitoring agent invocation counts and setting budget alerts.

2. **Amazon Redshift (+$196.16, +25.8%)** — Cluster costs increased, likely from scaling up for query workloads or additional concurrency. Review Redshift usage patterns and consider reserved nodes if usage is sustained.

3. **Bedrock Service (+$181.49, +558.6%)** — Foundation model usage grew dramatically. This could be from new model experimentation, increased token throughput, or switching to more expensive models. Review model selection and token usage.

### Top 3 Cost Decreases

1. **Amazon EC2 Compute (-$130.71, -15.6%)** — Instances were likely downsized, stopped, or migrated. Positive optimization signal.

2. **EC2 - Other (-$54.03, -12.0%)** — Reduced EBS volumes, data transfer, or elastic IPs. Aligns with EC2 compute reduction.

3. **Amazon DocumentDB (-$39.60, -74.4%)** — Significant reduction suggests instances were stopped or downsized. Verify this was intentional.

---

## Corrective Actions

| Priority | Action | Impact |
|----------|--------|--------|
| High | Set budget alert for Bedrock AgentCore at $200/month | Catch agent cost spikes early |
| High | Review Bedrock model selection — check if cheaper models can serve the same use case | Potential 50-80% reduction |
| Medium | Investigate CloudTrail cost doubling — check if data events were enabled unnecessarily | ~$120/month savings |
| Medium | Review Redshift usage — consider reserved nodes if sustained | ~$200/month savings potential |
| Low | Confirm DocumentDB reduction was intentional | Avoid service disruption |

---

## Methodology

- **Estimate source:** February 2026 actual spend (used as a proxy forecast for March, assuming stable infrastructure)
- **Actual source:** March 2026 Cost Explorer data via `getCostAndUsage` (UnblendedCost, grouped by SERVICE)
- **Variance calculation:** `(actual - estimate) / estimate × 100`
- **Accuracy score:** `1 - |total_variance| / total_actual × 100 = 87.6%`
- **Threshold logic:** >25% → INVESTIGATE, 10-25% → REVIEW, ≤10% → OK
- **Anomaly detection:** AWS Cost Anomaly Detection API — 0 anomalies found for the period
- **Forecast source:** AWS ML-based getCostForecast (Apr–Jun 2026)
- **Tools used:** `cost_explorer` (getCostAndUsage, getCostForecast), `cost_anomaly`, `session_sql`
