# MCP Server Setup

Add these three servers to your MCP configuration (`~/.kiro/settings/mcp.json` or `.kiro/settings/mcp.json`):

```json
{
  "mcpServers": {
    "awslabs.billing-cost-management-mcp-server": {
      "command": "uvx",
      "args": ["awslabs.billing-cost-management-mcp-server@latest"],
      "env": {
        "AWS_REGION": "us-east-1",
        "AWS_PROFILE": "default",
        "FASTMCP_LOG_LEVEL": "ERROR"
      },
      "disabled": false,
      "autoApprove": ["cost_explorer", "budgets", "cost_optimization", "compute_optimizer", "ri_performance", "sp_performance", "cost_anomaly", "cost_comparison", "free_tier_usage", "rec_details"]
    },
    "awslabs.aws-pricing-mcp-server": {
      "command": "uvx",
      "args": ["awslabs.aws-pricing-mcp-server@latest"],
      "env": {
        "AWS_REGION": "us-east-1",
        "AWS_PROFILE": "default",
        "FASTMCP_LOG_LEVEL": "ERROR"
      },
      "disabled": false,
      "autoApprove": ["get_pricing", "get_pricing_service_codes", "get_pricing_service_attributes", "get_pricing_attribute_values", "analyze_cdk_project", "analyze_terraform_project", "generate_cost_report"]
    },
    "awslabs.cloudwatch-mcp-server": {
      "command": "uvx",
      "args": ["awslabs.cloudwatch-mcp-server@latest"],
      "env": {
        "AWS_REGION": "us-east-1",
        "AWS_PROFILE": "default",
        "FASTMCP_LOG_LEVEL": "ERROR"
      },
      "disabled": false,
      "autoApprove": ["list_metrics", "get_metric_statistics", "get_metric_data", "describe_alarms", "list_dashboards", "get_dashboard", "describe_log_groups", "get_log_events", "get_query_results", "describe_metric_filters"]
    }
  }
}
```

## AWS Prerequisites

- AWS CLI configured with valid credentials (`aws configure` or `~/.aws/credentials`)
- Python 3.10+ and `uv` installed ([Install uv](https://docs.astral.sh/uv/getting-started/installation/))
- Cost Explorer enabled in your AWS account
- IAM permissions: `ce:*`, `budgets:*`, `pricing:*`, `cloudwatch:*`, `logs:*`

## Quick Test

After setup, try: "Show me my AWS costs for last month by service"
