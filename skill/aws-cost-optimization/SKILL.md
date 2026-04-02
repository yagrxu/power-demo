---
name: aws-cost-optimization
description: AWS cost optimization skill for business teams and developers. Guides spending analysis, optimization recommendations, pre-deployment cost estimation, forecast-vs-actual variance analysis, and cost-aware development using AWS Pricing, Billing & Cost Management, and CloudWatch MCP servers. Use when the user asks about AWS costs, budgets, pricing, rightsizing, savings plans, cost forecasting, or cost comparison.
---

# AWS Cost Optimization Skill

Route cost-related requests to the right workflow, then read the detailed guide file before making tool calls.

## Prerequisites

This skill requires three MCP servers. If not already configured, guide the user to set them up — full config is in [setup.md](setup.md).

| MCP Server | Purpose |
|------------|---------|
| `awslabs.billing-cost-management-mcp-server` | Cost Explorer, budgets, optimization, anomalies, session SQL |
| `awslabs.aws-pricing-mcp-server` | Pricing intelligence, IaC analysis, cost reports |
| `awslabs.cloudwatch-mcp-server` | Metrics, alarms, logs for cost-performance correlation |

## How to Use This Skill

1. Match the user's request to a scenario in the table below
2. **Read the linked workflow file** before making any tool calls
3. Follow the workflow's step-by-step instructions

## Scenario Detection → Workflow File

### Cost Analysis & Governance

| User Says | Read This File | Key Tools |
|-----------|---------------|-----------|
| "What are we spending?" / cost breakdown / spending analysis | [cloud-financial-management.md](workflows/cloud-financial-management.md) | `cost_explorer` |
| "Compare this month vs last" / month-over-month | [cloud-financial-management.md](workflows/cloud-financial-management.md) | `cost_comparison` |
| "Check our budgets" / budget status | [cloud-financial-management.md](workflows/cloud-financial-management.md) | `budgets` |
| "Any unusual spending?" / anomalies / spikes | [cloud-financial-management.md](workflows/cloud-financial-management.md) | `cost_anomaly` |
| "Who's spending what?" / chargeback / cost allocation | [expenditure-attribution.md](workflows/expenditure-attribution.md) | `cost_explorer` (tags) |

### Optimization & Rightsizing

| User Says | Read This File | Key Tools |
|-----------|---------------|-----------|
| "Where can we save?" / reduce costs / optimization | [consumption-model.md](workflows/consumption-model.md) | `cost_optimization`, `compute_optimizer` |
| "Rightsizing recommendations" / overprovisioned | [consumption-model.md](workflows/consumption-model.md) | `compute_optimizer`, `rec_details` |
| "RI/SP utilization?" / reserved capacity | [consumption-model.md](workflows/consumption-model.md) | `ri_performance`, `sp_performance` |
| "Should we use managed service X?" / TCO | [managed-services-optimization.md](workflows/managed-services-optimization.md) | `get_pricing`, `cost_explorer` |

### Pre-Deployment & Pricing

| User Says | Read This File | Key Tools |
|-----------|---------------|-----------|
| "How much will this cost?" / estimate CDK/Terraform | [developer-cost-optimization.md](workflows/developer-cost-optimization.md) | `analyze_cdk_project`, `get_pricing`, `generate_cost_report` |
| "Compare Lambda vs ECS pricing" / service comparison | [developer-cost-optimization.md](workflows/developer-cost-optimization.md) | `get_pricing` |
| "Regional pricing differences" | [developer-cost-optimization.md](workflows/developer-cost-optimization.md) | `get_pricing` (multi-region) |
| "Free tier usage?" | [developer-cost-optimization.md](workflows/developer-cost-optimization.md) | `free_tier_usage` |

### Forecast vs Actual

| User Says | Read This File | Key Tools |
|-----------|---------------|-----------|
| "Estimate vs actual?" / cost variance / did we stay on budget | [forecast-vs-actual.md](workflows/forecast-vs-actual.md) | `get_pricing` + `cost_explorer` + `session_sql` |
| "Validate our cost estimate" / accuracy | [forecast-vs-actual.md](workflows/forecast-vs-actual.md) | `cost_explorer`, `session_sql` |
| "Why did costs exceed the estimate?" | [forecast-vs-actual.md](workflows/forecast-vs-actual.md) | `cost_explorer`, `get_metric_data` |

### Efficiency & Measurement

| User Says | Read This File | Key Tools |
|-----------|---------------|-----------|
| "Cost per user/transaction" / efficiency metrics / ROI | [efficiency-measurement.md](workflows/efficiency-measurement.md) | `session_sql`, `cost_explorer` |
| "How efficient is our cloud spend?" | [efficiency-measurement.md](workflows/efficiency-measurement.md) | `compute_optimizer`, `cost_explorer` |

### Service-Specific Optimization

| User Mentions | Read This File |
|--------------|---------------|
| EC2, instances, compute | [services/compute-cost-optimization.md](workflows/services/compute-cost-optimization.md) |
| Lambda, serverless | [services/lambda-cost-optimization.md](workflows/services/lambda-cost-optimization.md) |
| ECS, EKS, containers, Fargate | [services/containers-cost-optimization.md](workflows/services/containers-cost-optimization.md) |
| Bedrock, foundation models | [services/bedrock-cost-optimization.md](workflows/services/bedrock-cost-optimization.md) |
| Bedrock agents | [services/bedrock-agents-cost-optimization.md](workflows/services/bedrock-agents-cost-optimization.md) |
| SageMaker, ML training | [services/sagemaker-cost-optimization.md](workflows/services/sagemaker-cost-optimization.md) |
| RDS, Aurora, database | [services/rds-aurora-cost-optimization.md](workflows/services/rds-aurora-cost-optimization.md) |
| DynamoDB | [services/dynamodb-cost-optimization.md](workflows/services/dynamodb-cost-optimization.md) |
| S3, EBS, storage | [services/storage-cost-optimization.md](workflows/services/storage-cost-optimization.md) |
| VPC, load balancer, networking | [services/networking-cost-optimization.md](workflows/services/networking-cost-optimization.md) |
| Redshift, data warehouse | [services/redshift-cost-optimization.md](workflows/services/redshift-cost-optimization.md) |
| OpenSearch | [services/opensearch-cost-optimization.md](workflows/services/opensearch-cost-optimization.md) |
| Graviton, ARM | [services/graviton-cost-optimization.md](workflows/services/graviton-cost-optimization.md) |
| Reserved Instances, Savings Plans | [services/reserved-instances-cost-optimization.md](workflows/services/reserved-instances-cost-optimization.md) |
| CloudWatch, monitoring costs | [services/monitoring-cost-optimization.md](workflows/services/monitoring-cost-optimization.md) |
| CloudTrail costs | [services/cloudtrail-cost-optimization.md](workflows/services/cloudtrail-cost-optimization.md) |

### Not Sure Which Tool?

Read [tool-selection-guide.md](workflows/tool-selection-guide.md) for comprehensive query-to-tool mapping.

## Critical Rules (Always Apply)

- **Pricing API order**: `get_pricing_service_codes` → `get_pricing_service_attributes` → `get_pricing_attribute_values` → `get_pricing`. Skipping steps causes filter failures.
- **Cost Explorer `end_date` is exclusive**: "2026-04-01" to "2026-05-01" = April only.
- **`cost_comparison` is month-to-month only**: Both periods must start on 1st of month.
- **`session_sql` schema/data must be arrays**: Not strings or dicts.
- **`getCostAndUsageWithResources`**: Limited to last 14 days.
- **Cost Optimization Hub**: Only available in us-east-1.
- **Always use `UnblendedCost`** unless user specifically asks for BlendedCost.
- **Always include `limit`** in log queries to avoid context overflow.
