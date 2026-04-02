# Forecast vs Actual Cost Analysis

This steering file provides end-to-end guidance for estimating AWS costs before deployment, tracking actual spend post-deployment, and comparing forecasted vs actual costs to improve cost predictability and accountability.

## Core Principle

Close the loop between pre-deployment cost estimates and post-deployment reality. Every deployment should have a cost expectation, and every cost expectation should be validated against actual spend.

## When to Load This Steering

Load this when the user needs to:
- Estimate costs before deploying infrastructure
- Compare estimated costs against actual AWS spend
- Track cost accuracy of previous estimates
- Build a forecast-to-actual feedback loop
- Validate architecture cost assumptions post-deployment
- Investigate why actual costs deviate from forecasts

## Workflow Overview

```
Phase 1: Pre-Deployment Estimate
  ├── Analyze IaC project (CDK/Terraform)
  ├── Query AWS Pricing API for unit costs
  ├── Generate cost estimate report
  └── Record estimate baseline

Phase 2: Deployment & Tagging
  ├── Tag resources with estimate metadata
  └── Deploy infrastructure

Phase 3: Post-Deployment Tracking
  ├── Query actual spend via Cost Explorer
  ├── Pull cost forecast from AWS
  └── Detect anomalies against expectations

Phase 4: Variance Analysis
  ├── Compare estimate vs actual
  ├── Identify cost drivers for deviations
  └── Feed learnings back into future estimates
```

---

## Phase 1: Pre-Deployment Cost Estimation

### Step 1: Analyze Infrastructure Code

Identify which AWS services your infrastructure uses before querying pricing.

**For CDK projects:**
```javascript
const cdkAnalysis = usePower("aws-cost-optimization", "awslabs.aws-pricing-mcp-server", "analyze_cdk_project", {
  "project_path": "/path/to/cdk-app"
})
// Returns: List of AWS services, constructs, and resource types in the project
```

**For Terraform projects:**
```javascript
const tfAnalysis = usePower("aws-cost-optimization", "awslabs.aws-pricing-mcp-server", "analyze_terraform_project", {
  "project_path": "/path/to/terraform-project"
})
// Returns: List of AWS services and resource declarations
```

### Step 2: Query Unit Pricing

Use the Pricing API to get current rates for each identified service. Always follow the discovery workflow.

**Discovery workflow (required order):**
```javascript
// 1. Find the correct service code
const serviceCodes = usePower("aws-cost-optimization", "awslabs.aws-pricing-mcp-server", "get_pricing_service_codes", {
  "filter": "lambda"
})

// 2. Discover filterable attributes
const attributes = usePower("aws-cost-optimization", "awslabs.aws-pricing-mcp-server", "get_pricing_service_attributes", {
  "service_code": "AWSLambda"
})

// 3. Get valid values for filters
const values = usePower("aws-cost-optimization", "awslabs.aws-pricing-mcp-server", "get_pricing_attribute_values", {
  "service_code": "AWSLambda",
  "attribute_names": ["group", "location"]
})

// 4. Query pricing with precise filters
const pricing = usePower("aws-cost-optimization", "awslabs.aws-pricing-mcp-server", "get_pricing", {
  "service_code": "AWSLambda",
  "region": "us-east-1",
  "output_options": {"pricing_terms": ["OnDemand"], "exclude_free_products": true}
})
```

**Multi-region comparison (for regional cost decisions):**
```javascript
const regionalPricing = usePower("aws-cost-optimization", "awslabs.aws-pricing-mcp-server", "get_pricing", {
  "service_code": "AmazonEC2",
  "region": ["us-east-1", "us-west-2", "eu-west-1"],
  "filters": [
    {"Field": "instanceType", "Value": "m5.large", "Type": "EQUALS"}
  ],
  "output_options": {"pricing_terms": ["OnDemand"], "product_attributes": ["instanceType", "location"]}
})
```

### Step 3: Build Usage Assumptions

Document your expected usage patterns. These become the basis for your estimate and the benchmark for variance analysis.

**Common assumption categories:**

| Category | Example Assumptions |
|----------|-------------------|
| Compute | 2x m5.large running 24/7, 730 hrs/month |
| Serverless | 1M Lambda invocations/month, 256MB memory, 200ms avg duration |
| Storage | 500GB S3 Standard, 10K PUT + 100K GET requests/month |
| Database | db.r5.large RDS Multi-AZ, 100GB storage, 1M IOPS |
| Data Transfer | 100GB/month outbound, 50GB inter-region |
| AI/ML | 500K Bedrock input tokens + 200K output tokens/month |

### Step 4: Generate Cost Estimate Report

Combine pricing data with usage assumptions into a documented estimate.

```javascript
const costEstimate = usePower("aws-cost-optimization", "awslabs.aws-pricing-mcp-server", "generate_cost_report", {
  "pricing_data": pricing, // from Step 2
  "service_name": "My Application Stack",
  "related_services": ["Lambda", "DynamoDB", "API Gateway", "S3"],
  "pricing_model": "ON DEMAND",
  "assumptions": [
    "1M API requests/month via API Gateway",
    "1M Lambda invocations, 256MB memory, 200ms avg duration",
    "50GB DynamoDB storage, 1000 WCU / 5000 RCU on-demand",
    "100GB S3 Standard storage",
    "Standard ON DEMAND pricing, no Savings Plans"
  ],
  "exclusions": [
    "Data transfer costs between regions",
    "CloudWatch logging and monitoring costs",
    "Free Tier benefits (estimate shows full cost)"
  ],
  "detailed_cost_data": {
    "services": {
      "AWS Lambda": {
        "usage": "1M invocations/month, 256MB, 200ms avg",
        "estimated_cost": "$3.40",
        "unit_pricing": {
          "requests": "$0.20 per 1M requests",
          "compute": "$0.0000166667 per GB-second"
        },
        "usage_quantities": {
          "requests": "1,000,000",
          "compute": "1M × 0.2s × 0.25GB = 50,000 GB-seconds"
        },
        "calculation_details": "$0.20 + $0.0000166667 × 50,000 = $3.40"
      }
    }
  },
  "output_file": "cost-estimate-my-app-2026-04.md",
  "format": "markdown"
})
```

### Step 5: Check Existing Workload Estimates (Optional)

If your organization uses the AWS Pricing Calculator, check for existing workload estimates.

```javascript
// List available workload estimates
const estimates = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "bcm_pricing_calc", {
  "operation": "list_workload_estimates",
  "status_filter": "VALID"
})

// Get details of a specific estimate
const estimateDetail = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "bcm_pricing_calc", {
  "operation": "get_workload_estimate",
  "identifier": "estimate-id-from-above"
})

// Get modeled usage lines within the estimate
const estimateUsage = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "bcm_pricing_calc", {
  "operation": "list_workload_estimate_usage",
  "identifier": "estimate-id-from-above"
})
```

---

## Phase 2: Deployment & Tagging Strategy

### Tag Resources for Cost Tracking

Apply tags during deployment that link resources back to the cost estimate. This is critical for Phase 3 comparison.

**Recommended tags for forecast-vs-actual tracking:**

| Tag Key | Example Value | Purpose |
|---------|--------------|---------|
| `CostEstimateId` | `est-myapp-2026-04` | Links resources to a specific estimate |
| `Project` | `my-application` | Groups resources by project |
| `Environment` | `production` | Distinguishes prod/dev/staging costs |
| `DeployDate` | `2026-04-01` | Marks when resources were deployed |
| `EstimatedMonthlyCost` | `$450` | Records the expected monthly cost |
| `Owner` | `team-platform` | Identifies cost accountability |

**CDK tagging example:**
```typescript
import { Tags } from 'aws-cdk-lib';

Tags.of(stack).add('CostEstimateId', 'est-myapp-2026-04');
Tags.of(stack).add('Project', 'my-application');
Tags.of(stack).add('Environment', 'production');
Tags.of(stack).add('EstimatedMonthlyCost', '450');
```

**Terraform tagging example:**
```hcl
locals {
  cost_tags = {
    CostEstimateId      = "est-myapp-2026-04"
    Project             = "my-application"
    Environment         = "production"
    EstimatedMonthlyCost = "450"
  }
}
```

---

## Phase 3: Post-Deployment Cost Tracking

### Step 1: Query Actual Spend

After at least one full billing cycle (ideally 30 days), pull actual costs filtered by the tags applied during deployment.

**By project tag:**
```javascript
const actualCosts = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "cost_explorer", {
  "operation": "getCostAndUsage",
  "start_date": "2026-04-01",
  "end_date": "2026-05-01",
  "granularity": "MONTHLY",
  "metrics": "[\"UnblendedCost\"]",
  "group_by": "[{\"Type\": \"DIMENSION\", \"Key\": \"SERVICE\"}]",
  "filter": "{\"Tags\": {\"Key\": \"Project\", \"Values\": [\"my-application\"], \"MatchOptions\": [\"EQUALS\"]}}"
})
```

**By cost estimate ID (more precise):**
```javascript
const actualByEstimate = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "cost_explorer", {
  "operation": "getCostAndUsage",
  "start_date": "2026-04-01",
  "end_date": "2026-05-01",
  "granularity": "DAILY",
  "metrics": "[\"UnblendedCost\"]",
  "group_by": "[{\"Type\": \"DIMENSION\", \"Key\": \"SERVICE\"}]",
  "filter": "{\"Tags\": {\"Key\": \"CostEstimateId\", \"Values\": [\"est-myapp-2026-04\"], \"MatchOptions\": [\"EQUALS\"]}}"
})
```

**Daily granularity for trend analysis:**
```javascript
const dailyTrend = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "cost_explorer", {
  "operation": "getCostAndUsage",
  "start_date": "2026-04-01",
  "end_date": "2026-05-01",
  "granularity": "DAILY",
  "metrics": "[\"UnblendedCost\"]",
  "filter": "{\"Tags\": {\"Key\": \"Project\", \"Values\": [\"my-application\"], \"MatchOptions\": [\"EQUALS\"]}}"
})
// Use daily data to spot cost ramp-up patterns and identify when costs stabilized
```

### Step 2: Pull AWS Cost Forecast

Compare your pre-deployment estimate against AWS's ML-based forecast (which uses actual usage patterns).

```javascript
const awsForecast = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "cost_explorer", {
  "operation": "getCostForecast",
  "metric": "UNBLENDED_COST",
  "granularity": "MONTHLY",
  "start_date": "2026-05-01",
  "end_date": "2026-08-01",
  "filter": "{\"Tags\": {\"Key\": \"Project\", \"Values\": [\"my-application\"], \"MatchOptions\": [\"EQUALS\"]}}"
})
// Returns: AWS's ML-based forecast for the next 3 months
// Compare this against your original pre-deployment estimate
```

### Step 3: Detect Cost Anomalies

Check if actual spend triggered any anomaly detection alerts.

```javascript
const anomalies = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "cost_anomaly", {
  "start_date": "2026-04-01",
  "end_date": "2026-05-01",
  "total_impact_start": 50
})
// Review anomalies — they may explain deviations from your estimate
```

### Step 4: Check Budget Performance

If a budget was set based on the estimate, check its status.

```javascript
const budgetStatus = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "budgets", {
  "budget_name": "my-application-budget"
})
// Compare actual vs budgeted amount (which should align with your estimate)
```

---

## Phase 4: Variance Analysis

### Step 1: Calculate Variance

Use `session_sql` to join estimate data with actual spend for structured comparison.

```javascript
// Load estimate data into session database
// Note: schema and data must be arrays (not strings/dicts)
const loadEstimate = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "session_sql", {
  "query": "CREATE TABLE IF NOT EXISTS estimates (project TEXT, service TEXT, estimated_monthly REAL, assumptions TEXT)",
  "schema": ["project", "service", "estimated_monthly", "assumptions"],
  "data": [
    ["my-application", "AWS Lambda", 3.40, "1M invocations, 256MB, 200ms"],
    ["my-application", "Amazon DynamoDB", 125.00, "50GB, 1000 WCU on-demand"],
    ["my-application", "Amazon API Gateway", 3.50, "1M REST API calls"],
    ["my-application", "Amazon S3", 2.30, "100GB Standard"]
  ],
  "table_name": "estimates"
})

// Query actual costs and store in session
// (After running getCostAndUsage, the data is available in session tables)

// Compare estimate vs actual
const varianceAnalysis = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "session_sql", {
  "query": `
    SELECT 
      e.service,
      e.estimated_monthly,
      a.actual_monthly,
      (a.actual_monthly - e.estimated_monthly) as variance_amount,
      ROUND((a.actual_monthly - e.estimated_monthly) / e.estimated_monthly * 100, 1) as variance_pct,
      e.assumptions,
      CASE 
        WHEN ABS((a.actual_monthly - e.estimated_monthly) / e.estimated_monthly * 100) > 25 THEN 'INVESTIGATE'
        WHEN ABS((a.actual_monthly - e.estimated_monthly) / e.estimated_monthly * 100) > 10 THEN 'REVIEW'
        ELSE 'OK'
      END as status
    FROM estimates e
    LEFT JOIN actuals a ON e.service = a.service AND e.project = a.project
    WHERE e.project = 'my-application'
    ORDER BY ABS(variance_amount) DESC
  `
})
```

### Step 2: Identify Cost Drivers for Deviations

For services with significant variance, drill into the cost drivers.

**Resource-level cost breakdown (last 14 days):**
```javascript
const resourceCosts = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "cost_explorer", {
  "operation": "getCostAndUsageWithResources",
  "start_date": "2026-04-17",
  "end_date": "2026-05-01",
  "granularity": "DAILY",
  "filter": "{\"And\": [{\"Tags\": {\"Key\": \"Project\", \"Values\": [\"my-application\"]}}, {\"Dimensions\": {\"Key\": \"SERVICE\", \"Values\": [\"Amazon DynamoDB\"]}}]}",
  "group_by": "[{\"Type\": \"DIMENSION\", \"Key\": \"RESOURCE_ID\"}]"
})
```

**Usage type breakdown (what exactly are you paying for):**
```javascript
const usageBreakdown = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "cost_explorer", {
  "operation": "getCostAndUsage",
  "start_date": "2026-04-01",
  "end_date": "2026-05-01",
  "granularity": "MONTHLY",
  "metrics": "[\"UnblendedCost\", \"UsageQuantity\"]",
  "group_by": "[{\"Type\": \"DIMENSION\", \"Key\": \"USAGE_TYPE\"}]",
  "filter": "{\"And\": [{\"Tags\": {\"Key\": \"Project\", \"Values\": [\"my-application\"]}}, {\"Dimensions\": {\"Key\": \"SERVICE\", \"Values\": [\"Amazon DynamoDB\"]}}]}"
})
// This reveals whether the deviation is from storage, read/write capacity, 
// data transfer, backups, or other usage types
```

### Step 3: Correlate with Performance Metrics

Use CloudWatch to understand whether cost deviations correlate with usage patterns.

```javascript
// Check if Lambda invocations exceeded estimate
const lambdaInvocations = usePower("aws-cost-optimization", "awslabs.cloudwatch-mcp-server", "get_metric_data", {
  "namespace": "AWS/Lambda",
  "metric_name": "Invocations",
  "start_time": "2026-04-01T00:00:00Z",
  "end_time": "2026-05-01T00:00:00Z",
  "statistic": "Sum",
  "dimensions": [{"name": "FunctionName", "value": "my-app-handler"}]
})
// Compare actual invocation count against the 1M assumed in the estimate

// Check DynamoDB consumed capacity vs provisioned
const dynamoCapacity = usePower("aws-cost-optimization", "awslabs.cloudwatch-mcp-server", "get_metric_data", {
  "namespace": "AWS/DynamoDB",
  "metric_name": "ConsumedWriteCapacityUnits",
  "start_time": "2026-04-01T00:00:00Z",
  "end_time": "2026-05-01T00:00:00Z",
  "statistic": "Sum",
  "dimensions": [{"name": "TableName", "value": "my-app-table"}]
})
```

### Step 4: Document Findings and Update Future Estimates

Record what you learned for improving future estimates.

**Variance report template:**

```markdown
# Cost Variance Report: my-application (April 2026)

## Summary
| Metric | Value |
|--------|-------|
| Estimated Monthly Cost | $134.20 |
| Actual Monthly Cost | $187.45 |
| Total Variance | +$53.25 (+39.7%) |
| Status | INVESTIGATE |

## Service-Level Variance
| Service | Estimated | Actual | Variance | Driver |
|---------|-----------|--------|----------|--------|
| DynamoDB | $125.00 | $172.30 | +$47.30 (+37.8%) | On-demand WCU 3x higher than assumed |
| Lambda | $3.40 | $5.15 | +$1.75 (+51.5%) | Avg duration 350ms vs 200ms assumed |
| API Gateway | $3.50 | $7.50 | +$4.00 (+114%) | 2.1M requests vs 1M assumed |
| S3 | $2.30 | $2.50 | +$0.20 (+8.7%) | Within tolerance |

## Root Causes
1. API traffic was 2.1x the assumed 1M requests/month
2. DynamoDB write patterns were bursty, driving higher on-demand costs
3. Lambda cold starts increased average duration beyond estimate

## Corrective Actions
1. Update traffic assumption to 2.5M requests/month for next estimate
2. Evaluate DynamoDB provisioned capacity mode for predictable write patterns
3. Consider Lambda provisioned concurrency to reduce cold start impact

## Estimate Accuracy Score: 71.5%
(Calculated as: 1 - |variance| / actual = 1 - 53.25/187.45)
```

---

## Common Variance Patterns

### Pattern 1: Traffic Exceeded Assumptions
**Symptom:** Actual costs 2-5x estimate across multiple services
**Root Cause:** Usage volume (requests, invocations, queries) was underestimated
**Detection:**
```javascript
// Compare actual request volume against estimate
const apiRequests = usePower("aws-cost-optimization", "awslabs.cloudwatch-mcp-server", "get_metric_data", {
  "namespace": "AWS/ApiGateway",
  "metric_name": "Count",
  "start_time": "2026-04-01T00:00:00Z",
  "end_time": "2026-05-01T00:00:00Z",
  "statistic": "Sum",
  "dimensions": [{"name": "ApiName", "value": "my-api"}]
})
```
**Fix:** Update traffic assumptions, consider auto-scaling cost implications, evaluate rate limiting

### Pattern 2: Data Transfer Costs Missed
**Symptom:** Unexpected line items for data transfer, actual exceeds estimate by fixed amount
**Root Cause:** Inter-region, inter-AZ, or internet egress transfer not included in estimate
**Detection:**
```javascript
const dataTransferCosts = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "cost_explorer", {
  "operation": "getCostAndUsage",
  "start_date": "2026-04-01",
  "end_date": "2026-05-01",
  "granularity": "MONTHLY",
  "metrics": "[\"UnblendedCost\"]",
  "group_by": "[{\"Type\": \"DIMENSION\", \"Key\": \"USAGE_TYPE\"}]",
  "filter": "{\"Dimensions\": {\"Key\": \"USAGE_TYPE\", \"Values\": [\"DataTransfer\"], \"MatchOptions\": [\"STARTS_WITH\"]}}"
})
```
**Fix:** Always include data transfer in estimates, use VPC endpoints to reduce costs

### Pattern 3: On-Demand vs Reserved Pricing Gap
**Symptom:** Estimate used on-demand pricing, but actual could be lower with commitments
**Root Cause:** Estimate didn't account for existing Savings Plans or RI coverage
**Detection:**
```javascript
// Check existing Savings Plans coverage
const spCoverage = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "sp_performance", {
  "operation": "get_savings_plans_coverage",
  "start_date": "2026-04-01",
  "end_date": "2026-05-01",
  "granularity": "MONTHLY"
})
```
**Fix:** Factor in existing commitment discounts, or flag on-demand pricing as worst-case in estimates

### Pattern 4: Storage Growth Underestimated
**Symptom:** Storage costs grow linearly month-over-month beyond estimate
**Root Cause:** Data accumulation rate was underestimated, no lifecycle policies
**Detection:**
```javascript
// Track storage growth trend
const storageTrend = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "cost_explorer", {
  "operation": "getCostAndUsage",
  "start_date": "2026-01-01",
  "end_date": "2026-05-01",
  "granularity": "MONTHLY",
  "metrics": "[\"UnblendedCost\", \"UsageQuantity\"]",
  "filter": "{\"And\": [{\"Tags\": {\"Key\": \"Project\", \"Values\": [\"my-application\"]}}, {\"Dimensions\": {\"Key\": \"SERVICE\", \"Values\": [\"Amazon Simple Storage Service\"]}}]}"
})
```
**Fix:** Include data growth rate in estimates, implement S3 lifecycle policies, use appropriate storage classes

### Pattern 5: Free Tier Expiration
**Symptom:** Costs jump significantly after 12 months
**Root Cause:** Estimate assumed Free Tier benefits that have since expired
**Detection:**
```javascript
const freeTierUsage = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "free_tier_usage", {
  "operation": "get_free_tier_usage"
})
// Check for services approaching or exceeding Free Tier limits
```
**Fix:** Always generate two estimates — one with Free Tier, one without — and note expiration dates

---

## Continuous Improvement: The Feedback Loop

### Monthly Variance Review Checklist

- [ ] Pull actual costs for all active projects with estimates
- [ ] Calculate per-service variance percentages
- [ ] Flag services with >25% variance for investigation
- [ ] Correlate cost deviations with CloudWatch usage metrics
- [ ] Check for cost anomaly alerts during the period
- [ ] Update assumption library with actual usage patterns
- [ ] Adjust future estimates based on learnings
- [ ] Document root causes for significant deviations

### Estimate Accuracy Tracking

Track estimate accuracy over time to improve your cost modeling.

```javascript
// Monthly accuracy tracking query
const accuracyTrend = usePower("aws-cost-optimization", "awslabs.billing-cost-management-mcp-server", "session_sql", {
  "query": `
    SELECT 
      month,
      project,
      estimated_total,
      actual_total,
      ROUND(ABS(actual_total - estimated_total) / actual_total * 100, 1) as error_pct,
      ROUND((1 - ABS(actual_total - estimated_total) / actual_total) * 100, 1) as accuracy_pct
    FROM monthly_variance_history
    ORDER BY month DESC, project
  `
})
```

**Accuracy targets:**
| Maturity Level | Accuracy Target | Typical Variance |
|---------------|----------------|-----------------|
| Starting out | >50% | ±50-100% |
| Developing | >70% | ±20-50% |
| Mature | >85% | ±10-20% |
| Advanced | >90% | ±5-10% |

### Assumption Library

Maintain a library of validated assumptions based on actual usage data. Update after each variance review.

| Assumption Category | Initial Estimate | Validated Value | Source |
|--------------------|-----------------|----------------|--------|
| Lambda avg duration | 200ms | 350ms | CloudWatch Apr 2026 |
| API monthly requests | 1M | 2.1M | CloudWatch Apr 2026 |
| DynamoDB WCU pattern | Steady 1000 | Bursty, peaks at 5000 | CloudWatch Apr 2026 |
| S3 storage growth | 10GB/month | 12GB/month | Cost Explorer trend |
| Data transfer egress | 50GB/month | 85GB/month | Cost Explorer usage type |

---

## Integration with Other Steering Files

- **Pre-deployment architecture decisions:** See [Developer Cost Optimization](./developer-cost-optimization.md) for cost-aware design patterns
- **Service-specific pricing deep dives:** See service guides in `./services/` for per-service optimization
- **Budget setup based on estimates:** See [Cloud Financial Management](./cloud-financial-management.md) for budget governance
- **Optimization after variance analysis:** See [Consumption Model](./consumption-model.md) for rightsizing and scaling strategies
- **Cost allocation for multi-project tracking:** See [Expenditure Attribution](./expenditure-attribution.md) for tagging and chargeback

---

## Quick Reference: Tool-to-Phase Mapping

| Phase | Tools Used | MCP Server |
|-------|-----------|------------|
| 1. Estimate | `analyze_cdk_project`, `analyze_terraform_project`, `get_pricing`, `get_pricing_service_codes`, `get_pricing_service_attributes`, `get_pricing_attribute_values`, `generate_cost_report` | aws-pricing |
| 1. Estimate (optional) | `bcm_pricing_calc` | billing-cost-management |
| 3. Track Actual | `cost_explorer` (getCostAndUsage), `cost_explorer` (getCostForecast), `cost_anomaly`, `budgets` | billing-cost-management |
| 3. Track Actual | CloudWatch metrics for usage correlation | cloudwatch |
| 4. Variance | `session_sql`, `cost_explorer` (getCostAndUsageWithResources), `cost_comparison` | billing-cost-management |
| 4. Variance | `get_metric_data` for usage vs assumption validation | cloudwatch |
