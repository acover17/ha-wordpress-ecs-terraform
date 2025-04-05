High Availability WordPress on ECS Fargate with Terraform
A production-ready, highly available WordPress deployment on AWS ECS Fargate, provisioned with Terraform, featuring CI/CD integration, security scanning, and cost control.

Table of Contents
Project Overview
Key Features
Architecture Overview
Prerequisites
Repository Structure
Setup Instructions
AWS Account Setup
Repository Setup
Terraform Deployment
WordPress Configuration
Usage Guide
Cost Management
Monitoring and Logging
Security Considerations
Backup & Disaster Recovery
Contributing Guidelines
License Information
Troubleshooting
Project Overview
This project provisions a production-grade WordPress deployment on AWS using Amazon ECS with Fargate for containerization, offering high availability across multiple Availability Zones. The entire infrastructure is defined as code using Terraform, enabling repeatable, version-controlled deployments with CI/CD integration via GitHub Actions.

The architecture follows AWS best practices for security, scalability, and reliability, featuring private subnet deployments, automated database backups, CDN integration, and comprehensive monitoring. It's designed to be cost-effective while maintaining production-level resilience and performance.

Key Features
High Availability
Multi-AZ deployment
Load-balanced ECS tasks
RDS Multi-AZ configuration
Security
Private subnet isolation
Security group restrictions
Secrets management
TLS encryption
DevOps Integration
Infrastructure as Code (Terraform)
CI/CD with GitHub Actions
Security scanning (tfsec)
Version-controlled configurations
Cost Optimization
AWS Budgets integration
NAT Instance vs. NAT Gateway savings
CloudWatch cost alarms
Right-sized resources
Architecture Overview
[Architecture Diagram Placeholder]
Add your architecture diagram here after creation with diagrams.net

Infrastructure Components
AWS Services
VPC & Subnets
Security Groups
NAT Instance (t2/t3.micro)
Route 53
ECS (Fargate)
RDS (Multi-AZ)
S3 (State/Backups)
Secrets Manager
Parameter Store
CloudFront
CloudWatch
IAM
Application Load Balancer
AWS Budgets
External Tools
Terraform (IaC)
Git/GitHub
GitHub Actions (CI/CD)
tfsec (Security Scanning)
Docker
Diagrams.net
Grafana Cloud (Free Tier)
Architecture Details
The architecture follows a tiered design with components distributed across multiple Availability Zones:

Component	Location	Purpose
Route 53	Global	DNS routing
CloudFront	Edge Locations	Content delivery and caching
ALB	Public Subnets (Multi-AZ)	Load balancing
NAT Instance	Public Subnet	Outbound internet access for private resources
ECS Fargate Tasks	Private Subnets (Multi-AZ)	WordPress application containers
RDS Instances	Private Subnets (Multi-AZ)	MySQL database (Primary & Standby)
Traffic Flow
User → Route 53 → CloudFront
CloudFront → Application Load Balancer
ALB → ECS Fargate Tasks (WordPress)
ECS Tasks ↔ RDS (database queries)
ECS Tasks → NAT Instance → Internet (for updates)
Prerequisites
AWS Account Requirements
AWS Account with administrator access
AWS CLI installed and configured
Registered domain in Route 53 (recommended)
Increased service limits if needed
Development Environment
Terraform CLI (v1.0.0+)
Git client
GitHub account
Docker (for local testing, optional)
 Note: For security and cost-control reasons, it's recommended to use a dedicated AWS account for this project or implement strict IAM policies to limit resource creation.
Repository Structure

/
├── .github/workflows/     # GitHub Actions CI/CD workflows
│   ├── terraform-plan.yml
│   ├── terraform-apply.yml
│   └── security-scan.yml
├── docs/                  # Documentation
│   ├── architecture.png   # Architecture diagram
│   ├── setup-guide.md     # Detailed setup instructions
│   └── cost-analysis.md   # Cost considerations and optimization
├── modules/               # Reusable Terraform modules
│   ├── networking/        # VPC, subnets, routing, security groups
│   ├── ecs/               # ECS cluster, service, task definitions
│   ├── rds/               # RDS instances, parameter groups, subnet groups
│   ├── alb/               # Application Load Balancer, target groups
│   ├── cloudfront/        # CloudFront distribution
│   ├── dns/               # Route 53 configurations
│   ├── security/          # IAM, Secrets Manager, Parameter Store
│   └── monitoring/        # CloudWatch resources, AWS Budgets
├── environments/          # Environment-specific configurations
│   ├── dev/
│   ├── staging/
│   └── prod/
├── scripts/               # Utility scripts
│   ├── backup.sh
│   ├── cost-report.sh
│   └── security-check.sh
├── .gitignore
├── LICENSE                # MIT License
└── README.md              # This file
                    
Setup Instructions
AWS Account Setup
1. Configure AWS Budget
To limit project costs to $20 and automatically pause when this threshold is reached:

Log in to the AWS Management Console
Navigate to AWS Budgets (under AWS Cost Management)
Click "Create budget"
Choose "Cost budget" and click "Next"
Set up budget details:
Name: "WordPress-Project-Budget"
Period: Monthly
Start date: Current month
Budget amount: $20
Configure alerts at 80% and 100% thresholds with email notifications
Setup automated actions to apply an IAM policy that restricts resource creation when the budget is exceeded
Sample IAM Policy for Budget Actions
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "ecs:RunTask",
        "ecs:StartTask",
        "ecs:UpdateService",
        "rds:CreateDBInstance",
        "ec2:RunInstances"
      ],
      "Resource": "*"
    }
  ]
}
2. Configure AWS CLI
# Configure AWS CLI with your credentials
aws configure

# Verify configuration
aws sts get-caller-identity
3. Create S3 Bucket for Terraform State
# Create S3 bucket for Terraform state
aws s3api create-bucket \
    --bucket your-terraform-state-bucket \
    --region us-east-1

# Enable versioning on the bucket
aws s3api put-bucket-versioning \
    --bucket your-terraform-state-bucket \
    --versioning-configuration Status=Enabled
Repository Setup
1. Clone Repository
git clone https://github.com/yourusername/ha-wordpress-ecs-terraform.git
cd ha-wordpress-ecs-terraform
2. Configure GitHub Secrets
For CI/CD workflows, add the following secrets to your GitHub repository:

AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
 Security Warning: Use IAM roles with limited permissions for CI/CD automation. Never use root credentials.
Terraform Deployment
1. Initialize Terraform
# Navigate to your environment directory
cd environments/dev

# Initialize Terraform with the S3 backend
terraform init \
  -backend-config="bucket=your-terraform-state-bucket" \
  -backend-config="key=wordpress/dev/terraform.tfstate" \
  -backend-config="region=us-east-1"
2. Configure Variables
Create a terraform.tfvars file with your configuration values:

# terraform.tfvars

# General
project_name    = "wordpress"
environment     = "dev"
aws_region      = "us-east-1"

# VPC
vpc_cidr        = "10.0.0.0/16"
az_count        = 2

# Domain
domain_name     = "example.com"
subdomain       = "blog"

# RDS
db_name         = "wordpress"
db_instance     = "db.t3.small"
multi_az        = true

# ECS
ecs_task_cpu    = "512"
ecs_task_memory = "1024"
min_capacity    = 2
max_capacity    = 4

# Budget
budget_limit    = 20
3. Plan Deployment
terraform plan -out=tfplan
4. Apply Configuration
terraform apply tfplan
 Note: Initial deployment may take 15-20 minutes, especially for RDS provisioning.
WordPress Configuration
1. Access WordPress Setup
After successful deployment, access the WordPress setup page:

https://blog.example.com/wp-admin/setup-config.php
2. Complete Initial Setup
Follow the WordPress installation wizard
Create admin account
Configure basic site settings
3. Install Essential Plugins
Recommended plugins for high-availability WordPress:

W3 Total Cache (performance)
Wordfence Security (security)
UpdraftPlus (backups)
WP Offload Media (S3 integration for media files)
Usage Guide
Everyday Operations
Scaling WordPress
The deployment uses ECS Auto Scaling which automatically adjusts the number of WordPress containers based on load. To manually adjust the scaling parameters:

# Update min/max capacity in terraform.tfvars
min_capacity    = 3  # Minimum containers
max_capacity    = 6  # Maximum containers

# Apply changes
terraform apply
Database Maintenance
RDS performs automatic backups according to the retention period specified in your configuration. To manually create a DB snapshot:

aws rds create-db-snapshot \
    --db-instance-identifier wordpress-db \
    --db-snapshot-identifier wordpress-manual-backup-$(date +%Y-%m-%d)
Content Management
Access the WordPress admin dashboard to manage content:

https://blog.example.com/wp-admin/
 Note: When using a Multi-AZ deployment with ECS Fargate, all content updates through the WordPress admin interface are automatically synchronized across instances.
Cost Management
This project is designed with cost optimization in mind while maintaining high availability.

Cost Optimization Features
NAT Instance vs NAT Gateway: Using a t3.micro NAT Instance (~$7.45/month) instead of a NAT Gateway (~$32/month + data transfer)
AWS Budgets: Configured to limit monthly spending to $20 with automated actions
CloudWatch Alarms: Set up to monitor and alert on cost anomalies
Right-sized Resources: Appropriately sized ECS tasks and RDS instances
Estimated Monthly Costs
Service	Configuration	Est. Monthly Cost
NAT Instance	t3.micro	$7.45
RDS	db.t3.small (Multi-AZ)	$51.00
ECS Fargate	2 × 0.5 vCPU, a1 GB	$21.08
ALB	Standard	$16.43
Route 53	Hosted Zone + Queries	$0.50
S3 + CloudFront	Minimal usage	$1.00
CloudWatch	Basic monitoring	$2.00
Other	Data transfer, etc.	$1.00
TOTAL (estimated)	$100.46
 Note: The full high-availability configuration exceeds the $20 budget. For a strict $20 budget, consider these cost-reduction options:
Use single-AZ for RDS (-$25.50)
Reduce to 1 ECS Fargate task (-$10.54)
Use a smaller RDS instance type like db.t3.micro (-$25.00)
Consider using EC2 instead of Fargate for containerization
Monitoring and Logging
CloudWatch Integration
The deployment includes CloudWatch monitoring for all AWS services:

ECS Task metrics (CPU, memory, network)
RDS performance metrics
ALB request counts, latency, errors
CloudFront distribution metrics
Cost and usage metrics
Key CloudWatch Alarms
Alarm	Metric	Threshold	Action
High CPU Utilization	ECS Service CPU Usage	> 80% for 5 minutes	Scale up ECS service
Database High CPU	RDS CPU Utilization	> 85% for 10 minutes	Send notification
HTTP Errors	ALB 5XX Error Rate	> 5% for 5 minutes	Send notification
Budget Alert	Estimated Charges	> $16 (80% of budget)	Send notification
Grafana Integration (Optional)
For enhanced visualization, the project includes optional integration with Grafana Cloud (Free Tier):

Sign up for Grafana Cloud free tier
Configure CloudWatch as a data source
Import WordPress monitoring dashboards
Set up additional alerting rules
Security Considerations
Security Features
Network Security
Private subnet isolation for application and database
Security groups with least privilege access
HTTPS enforcement via CloudFront and ALB
WAF integration for OWASP Top 10 protection (optional)
Data Security
RDS encryption at rest
S3 bucket encryption
TLS for all data in transit
Secrets stored in AWS Secrets Manager
Identity & Access
IAM roles with least privilege
Task execution roles for ECS
Secure parameter access via Parameter Store
WordPress admin access restricted by IP (optional)
Compliance & Scanning
tfsec scans in CI/CD pipeline
ECR image scanning for container vulnerabilities
Automatic WordPress core and plugin updates
CloudTrail for API activity logging
WordPress Security Hardening
Beyond infrastructure security, implement these WordPress-specific hardening measures:

Install security plugins (Wordfence, Sucuri, etc.)
Use strong admin passwords and change default admin username
Implement two-factor authentication for admin access
Keep WordPress core, themes, and plugins updated
Limit login attempts and implement CAPTCHA
Disable file editing in the WordPress admin
Regularly scan for malware and vulnerabilities
Backup & Disaster Recovery
Backup Strategy
The deployment includes a comprehensive backup strategy:

Component	Backup Method	Frequency	Retention
Database (RDS)	Automated RDS Snapshots	Daily	7 days
WordPress Files	EFS Backups to S3	Daily	30 days
Media Files	S3 Versioning	Continuous	Indefinite
Infrastructure Code	Git Repository	Per commit	Indefinite
Disaster Recovery Process
Database Recovery: Restore from the latest RDS snapshot
WordPress Files: Restore from S3 backups to EFS
Infrastructure: Redeploy using Terraform code
DNS: Update Route 53 records if necessary
Validation: Verify WordPress functionality and data integrity
Recovery Time Objectives (RTO)
Infrastructure rebuild: ~30-45 minutes
Database restoration: ~15-30 minutes
WordPress files restoration: ~10-20 minutes
Total RTO: ~1-2 hours
Contributing Guidelines
Contributions to this project are welcome! Please follow these guidelines:

Development Workflow
Fork the repository
Create a feature branch (git checkout -b feature/amazing-feature)
Make your changes
Run terraform fmt to format your code
Run terraform validate to validate your changes
Run tfsec to check for security issues
Commit your changes (git commit -m 'Add some amazing feature')
Push to the branch (git push origin feature/amazing-feature)
Open a Pull Request
Coding Standards
Follow HashiCorp's Terraform Style Guide
Document all module inputs and outputs
Use meaningful variable and resource names
Include descriptions for all variables
Group related resources together
Use consistent naming conventions
License Information
This project is licensed under the MIT License - see the LICENSE file for details.

MIT License

Copyright (c) 2025 Antonio Cover

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
Troubleshooting
Terraform Apply Fails with State Lock Error
If terraform apply fails with a state lock error, the previous operation might not have cleanly released the state lock.

terraform force-unlock [LOCK_ID]
WordPress Cannot Connect to Database
Check the following:

Security group rules allow traffic from ECS tasks to RDS on port 3306
Database credentials in Secrets Manager are correct
RDS instance is in "available" state
CloudFront Returns 503 Error
Verify the following:

ALB health checks are passing for ECS tasks
CloudFront distribution is enabled and deployed
Origin (ALB) is responding properly
ECS Tasks Failing to Start
Check CloudWatch Logs for the ECS tasks to identify startup issues. Common problems include:

Insufficient memory or CPU allocation
Missing permissions in task execution role
Unable to fetch secrets from Secrets Manager
Database connection errors
Budget Exceeded and Resources Stopped
If AWS Budget actions have stopped resource creation:

Review current costs in AWS Cost Explorer
Identify high-cost resources
Optimize or remove unnecessary resources
Adjust budget or temporarily disable budget actions
Conclusion
This project provides a robust, scalable, and secure WordPress deployment using AWS's managed services and best practices for high availability. By leveraging Terraform for infrastructure as code, CI/CD for automated deployments, and comprehensive monitoring, you can maintain a reliable WordPress environment with minimal operational overhead.

For questions, issues, or contributions, please use the GitHub repository's issue tracker or submit pull requests according to the contributing guidelines.