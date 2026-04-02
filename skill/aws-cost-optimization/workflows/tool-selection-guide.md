---
inclusion: manual
---

# AWS Cost Optimization Tool Selection Guide

Use this guide to select the most appropriate tools based on user queries for both business and developer workflows.

## 🎯 Service Coverage Overview

This power includes **20 comprehensive service-specific optimization guides** covering:
- **5 Compute Services:** EC2, Lambda, Containers, AMD, Graviton
- **4 AI/ML Services:** Bedrock, Bedrock Agents, SageMaker, AI Workloads ← **NEW**
- **4 Database Services:** RDS/Aurora, DynamoDB, Redshift, OpenSearch
- **2 Storage Services:** Storage, FSx NetApp ONTAP
- **2 Security Services:** Security, CloudTrail
- **2 Management Services:** Monitoring, Reserved Instances
- **1 Networking Service:** VPC, Load Balancers
- **1 Analytics Service:** ElastiCache Redis
- **1 Application Service:** SQS

For service-specific queries, refer to the detailed guides in the [Service-Specific Optimization Queries](#service-specific-optimization-queries) section below.

## Query Pattern → Tool Mapping

### Business & FinOps Workflows

#### Cost Analysis Queries
- "show costs", "cost breakdown", "spending analysis" → `cost_explorer`
- "forecast", "predict costs", "future spending" → `cost_explorer` (getCostForecast)
- "compare costs", "month over month" → `cost_comparison`

#### Optimization Queries  
- "save money", "reduce costs", "optimization" → `cost_optimization`
- "rightsizing", "instance recommendations" → `compute_optimizer`
- "unused resources", "idle instances" → `cost_optimization`

#### Budget & Monitoring
- "budget", "spending limits", "alerts" → `budgets`
- "anomaly", "unusual spending", "spikes" → `cost_anomaly`
- "free tier", "free usage" → `free_tier_usage`
- "metrics", "CloudWatch metrics", "performance monitoring" → CloudWatch MCP tools (see service guides)
- "alarms", "monitoring alerts", "thresholds" → CloudWatch MCP tools (see service guides)
- "dashboards", "cost dashboards", "monitoring dashboards" → CloudWatch MCP tools (see service guides)

#### Reserved Capacity
- "reserved instances", "RI utilization" → `ri_performance`  
- "savings plans", "SP coverage" → `sp_performance`

### Developer & Engineering Workflows

#### Pre-Deployment Cost Analysis
- "CDK costs", "estimate CDK stack" → `analyze_cdk_project` + `get_pricing`
- "Terraform costs", "estimate Terraform" → `analyze_terraform_project` + `get_pricing`
- "architecture costs", "design costs" → `get_pricing` + `generate_cost_report`
- "before deployment", "pre-deploy analysis" → `analyze_cdk_project` or `analyze_terraform_project`

#### Service & Architecture Comparison
- "Lambda vs ECS", "service comparison" → `get_pricing` (multi-service)
- "cheapest option", "lowest cost" → `get_pricing` (with cost optimization filters)
- "regional pricing", "region comparison" → `get_pricing` (multi-region)
- "serverless costs", "container costs" → `get_pricing` (service-specific)
- "AI model comparison", "Bedrock vs SageMaker" → See [AI Workloads Cost Optimization](./services/ai-workloads-cost-optimization.md)
- "training vs inference costs", "ML cost modeling" → See [SageMaker Cost Optimization](./services/sagemaker-cost-optimization.md)

#### Development Environment Optimization
- "dev environment costs", "development spending" → `cost_explorer` (filtered by Environment=dev tags)
- "test environment", "staging costs" → `cost_explorer` (filtered by environment tags)
- "free tier usage", "avoid charges" → `free_tier_usage`
- "development budget", "team spending" → `budgets` + `cost_explorer`

#### Forecast vs Actual Analysis
- "estimate vs actual", "forecast vs actual" → See [Forecast vs Actual](./forecast-vs-actual.md) workflow
- "cost accuracy", "estimate accuracy" → `session_sql` + `cost_explorer`
- "why did costs exceed estimate", "cost deviation" → `cost_explorer` (getCostAndUsageWithResources) + CloudWatch
- "validate cost estimate", "check estimate" → `cost_explorer` (getCostAndUsage with tag filter)
- "cost variance", "budget variance" → `cost_comparison` + `budgets`
- "workload estimate", "pricing calculator" → `bcm_pricing_calc`

#### Cost-Aware Development
- "pricing intelligence", "service pricing" → `get_pricing`
- "cost modeling", "estimate costs" → `get_pricing` + `generate_cost_report`
- "architecture decision", "cost comparison" → `get_pricing` (comparative analysis)
- "development ROI", "cost per feature" → `cost_explorer` (custom analysis)
- "AI development costs", "ML project budgeting" → See [AI Workloads Cost Optimization](./services/ai-workloads-cost-optimization.md)
- "agent development costs", "Bedrock agent budgeting" → See [Bedrock Agents Cost Optimization](./services/bedrock-agents-cost-optimization.md)

#### Infrastructure Cost Analysis
- "Bedrock patterns", "AI costs" → `get_bedrock_patterns` + `get_pricing`
- "Bedrock agents", "agent costs", "agent optimization" → See [Bedrock Agents Cost Optimization](./services/bedrock-agents-cost-optimization.md)
- "AI workloads", "ML costs", "AI optimization" → See [AI Workloads Cost Optimization](./services/ai-workloads-cost-optimization.md)
- "SageMaker costs", "ML training", "inference costs" → See [SageMaker Cost Optimization](./services/sagemaker-cost-optimization.md)
- "storage optimization", "S3 costs" → `storage_lens` + `get_pricing`
- "database costs", "RDS vs DynamoDB" → `get_pricing` (service comparison)
- "compute costs", "EC2 vs Lambda" → `get_pricing` + `compute_optimizer`

### Service-Specific Optimization Queries

#### AI/ML Services
- "Bedrock costs", "foundation models", "token optimization" → See [Bedrock Cost Optimization](./services/bedrock-cost-optimization.md)
- "Bedrock agents", "agent workflows", "prompt caching" → See [Bedrock Agents Cost Optimization](./services/bedrock-agents-cost-optimization.md)
- "SageMaker optimization", "ML training costs", "inference optimization" → See [SageMaker Cost Optimization](./services/sagemaker-cost-optimization.md)
- "AI workload optimization", "GPU costs", "ML infrastructure" → See [AI Workloads Cost Optimization](./services/ai-workloads-cost-optimization.md)

#### Compute Services
- "EC2 optimization", "instance rightsizing", "compute costs" → See [Compute Cost Optimization](./services/compute-cost-optimization.md)
- "Lambda optimization", "serverless costs" → See [Lambda Cost Optimization](./services/lambda-cost-optimization.md)
- "container costs", "ECS/EKS optimization" → See [Containers Cost Optimization](./services/containers-cost-optimization.md)
- "Graviton optimization", "ARM processors" → See [Graviton Cost Optimization](./services/graviton-cost-optimization.md)

#### Database Services
- "RDS optimization", "Aurora costs" → See [RDS/Aurora Cost Optimization](./services/rds-aurora-cost-optimization.md)
- "DynamoDB optimization", "NoSQL costs" → See [DynamoDB Cost Optimization](./services/dynamodb-cost-optimization.md)
- "Redshift optimization", "data warehouse costs" → See [Redshift Cost Optimization](./services/redshift-cost-optimization.md)
- "OpenSearch optimization", "search costs" → See [OpenSearch Cost Optimization](./services/opensearch-cost-optimization.md)

#### Storage Services
- "storage optimization", "S3 costs", "EBS optimization" → See [Storage Cost Optimization](./services/storage-cost-optimization.md)
- "FSx optimization", "NetApp ONTAP costs" → See [FSx NetApp ONTAP Cost Optimization](./services/fsx-netapp-ontap-cost-optimization.md)

#### Networking Services
- "networking costs", "VPC optimization", "load balancer costs" → See [Networking Cost Optimization](./services/networking-cost-optimization.md)

#### Security Services
- "security costs", "KMS optimization", "GuardDuty costs" → See [Security Cost Optimization](./services/security-cost-optimization.md)
- "CloudTrail optimization", "logging costs" → See [CloudTrail Cost Optimization](./services/cloudtrail-cost-optimization.md)

#### Management Services
- "monitoring costs", "CloudWatch optimization" → See [Monitoring Cost Optimization](./services/monitoring-cost-optimization.md)
- "Reserved Instances", "RI optimization", "Savings Plans" → See [Reserved Instances Cost Optimization](./services/reserved-instances-cost-optimization.md)

#### Application Services
- "SQS optimization", "message queue costs" → See [SQS Cost Optimization](./services/sqs-cost-optimization.md)
- "ElastiCache optimization", "Redis costs" → See [ElastiCache Redis Cost Optimization](./services/elasticache-redis-cost-optimization.md)

### Cross-Functional Workflows

#### Pricing Intelligence
- "pricing", "cost comparison", "cheapest" → `get_pricing`
- "service codes", "available services" → `get_pricing_service_codes`
- "pricing attributes", "filter options" → `get_pricing_service_attributes`
- "valid values", "pricing options" → `get_pricing_attribute_values`

#### Comprehensive Analysis
- "detailed cost report", "full analysis" → `generate_cost_report`
- "cost recommendations", "optimization report" → `rec_details`
- "historical pricing", "bulk data" → `get_price_list_urls`

## Advanced Tools (Manual Approval)
- S3 storage analysis → `storage_lens`
- Custom SQL analysis → `session_sql`
- Pricing calculator → `bcm_pricing_calc`

## CloudWatch MCP Tools (Auto-Approved)
- Metrics analysis → `list_metrics`, `get_metric_statistics`, `get_metric_data`
- Alarm management → `describe_alarms`
- Dashboard management → `list_dashboards`, `get_dashboard`
- Log analysis → `describe_log_groups`, `get_log_events`, `start_query`
- Metric filters → `describe_metric_filters`

*Note: CloudWatch tools are integrated throughout service-specific guides for cost-performance correlation analysis.*

## Workflow Combinations

### Complete Pre-Deployment Analysis
1. `analyze_cdk_project` or `analyze_terraform_project` → Identify services
2. `get_pricing` → Get pricing for identified services
3. `generate_cost_report` → Generate comprehensive cost analysis

### Architecture Decision Support
1. `get_pricing` → Compare service options
2. `get_pricing_service_attributes` → Understand pricing dimensions
3. `get_pricing_attribute_values` → Explore configuration options

### Development Cost Optimization
1. `cost_explorer` → Analyze current development spending
2. `free_tier_usage` → Check Free Tier utilization
3. `cost_optimization` → Find optimization opportunities
4. `budgets` → Monitor spending limits

### AI/ML Cost Optimization Workflow
1. `cost_explorer` → Analyze current AI/ML spending by service
2. `get_pricing` → Compare model and instance pricing options
3. Refer to service-specific guides:
   - [AI Workloads Cost Optimization](./services/ai-workloads-cost-optimization.md) for comprehensive AI strategy
   - [Bedrock Agents Cost Optimization](./services/bedrock-agents-cost-optimization.md) for agent-specific optimization
   - [SageMaker Cost Optimization](./services/sagemaker-cost-optimization.md) for ML training and inference
   - [Bedrock Cost Optimization](./services/bedrock-cost-optimization.md) for foundation model optimization

### Service-Specific Optimization Workflow
1. `cost_explorer` → Identify high-cost services
2. Refer to appropriate service guide from the 20 available guides in `/services/`
3. `get_pricing` → Compare optimization options
4. CloudWatch MCP tools → Monitor performance metrics for cost correlation
5. `cost_optimization` → Implement recommendations
6. `budgets` → Monitor optimization results

### Performance-Cost Correlation Analysis
1. `cost_explorer` → Analyze service costs
2. CloudWatch MCP tools → Get performance metrics (`get_metric_statistics`, `get_metric_data`)
3. Service-specific guides → Apply optimization strategies
4. CloudWatch MCP tools → Set up monitoring (`describe_alarms`, `get_dashboard`)
5. `budgets` → Track cost impact of optimizations

### Forecast vs Actual Cost Analysis
1. `analyze_cdk_project` or `analyze_terraform_project` → Identify services in IaC
2. `get_pricing` → Get unit pricing for identified services
3. `generate_cost_report` → Document pre-deployment cost estimate
4. (Optional) `bcm_pricing_calc` → Check existing workload estimates
5. Deploy with cost-tracking tags (CostEstimateId, Project, Environment)
6. `cost_explorer` (getCostAndUsage) → Pull actual spend by tags
7. `cost_explorer` (getCostForecast) → Compare AWS forecast vs your estimate
8. `cost_anomaly` → Detect unexpected deviations
9. `session_sql` → Join estimate data with actuals for variance analysis
10. CloudWatch MCP tools → Correlate cost deviations with usage metrics

*Call action "readSteering" with powerName="aws-cost-optimization", steeringFile="forecast-vs-actual.md" for the full workflow.*