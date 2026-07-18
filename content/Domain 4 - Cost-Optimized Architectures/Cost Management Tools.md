# AWS Cost Management Tools

## AWS Cost Explorer
- Visualize and analyze costs and usage over time
- Forecast future costs (up to 12 months)
- Identify cost drivers
- Filter by service, account, tag, etc.
- Right-sizing recommendations for EC2
- Savings Plans recommendations

## AWS Budgets
- Set custom budgets with alerts
- Budget types: Cost, Usage, Reservation, Savings Plans
- Alert thresholds: actual or forecasted
- Actions: apply SCPs, stop instances, IAM policies
- Monthly, quarterly, or annual budgets

## AWS Cost and Usage Report (CUR)
- Most comprehensive cost data
- Hourly/daily line items for each service
- Delivered to S3 bucket
- Integrate with Athena, Redshift, or QuickSight for analysis
- Includes resource-level detail

## Savings Plans
- Flexible pricing model for compute
- 1 or 3-year hourly spend commitment
- **Compute Savings Plans** — most flexible (EC2, Fargate, Lambda)
- **EC2 Instance Savings Plans** — specific family + region
- Up to 72% savings vs On-Demand

## AWS Trusted Advisor
- Real-time guidance across 5 categories:
  - Cost Optimization
  - Performance
  - Security
  - Fault Tolerance
  - Service Limits
- Full checks require Business/Enterprise Support plan
- Free checks: S3 bucket permissions, Security Groups, MFA on root, EBS snapshots, service limits

## AWS Compute Optimizer
- ML-based recommendations for right-sizing
- Analyzes: EC2, ASG, Lambda, EBS volumes, ECS on Fargate
- Recommends optimal instance types based on utilization
- Requires at least 14 days of metrics

## AWS Organizations - Consolidated Billing
- Single bill for all accounts
- Volume pricing discounts (S3, EC2)
- Reserved Instance sharing across accounts

## Exam Tips
- Cost Explorer for visualization and forecasting
- Budgets for alerting and automated actions
- CUR for granular analysis (Athena queries)
- Trusted Advisor for quick wins (low-hanging fruit)
- Compute Optimizer for right-sizing recommendations
- Use tags for cost allocation and tracking

## SAA Decision Addendum

### Choose This When
- **Cost Explorer**: investigate trends, forecast spend, or view service/account breakdowns.
- **AWS Budgets**: alert when actual or forecasted spend crosses a threshold.
- **Cost and Usage Report**: detailed billing data is needed in S3 for Athena/Redshift/BI analysis.
- **Compute Optimizer**: rightsizing recommendations are needed for compute and related resources.
- **Consolidated billing**: multiple accounts should share billing, volume discounts, and centralized payment.

### Exam Traps
- Budgets alert; they do not automatically stop resources unless paired with automation.
- CUR is the most detailed billing dataset but requires analysis tooling.
- Cost allocation tags must be activated before they appear in billing reports.
- Trusted Advisor cost checks depend on support plan and are recommendation-focused, not enforcement.

---

Related: [[EC2 Pricing]] | [[S3 Cost Optimization]]
#aws #cost-management #tools #domain4
