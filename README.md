# Sembix Studio Deployment Template

This repository template provides a production-ready deployment pipeline for Sembix Studio using GitHub Actions and Terraform. Use this template to quickly set up automated deployments to AWS for your Sembix Studio instances.

## Overview

This template includes pre-configured GitHub Actions workflows that handle the full lifecycle of Sembix Studio deployments:

- **Automated Deployments**: Deploy Sembix Studio with a single click
- **Infrastructure as Code**: Leverages Terraform for consistent, reproducible infrastructure
- **Environment Management**: Support for multiple environments (dev, staging, prod, customer-specific)
- **Safety Controls**: Built-in approval gates and validation steps
- **Flexible Configuration**: Extensive customization options for networking, security, and resources

## Features

- Automated deployment workflow with validation and approval gates
- First-time bootstrap setup for new environments
- Database migration management
- UI deployment to S3/CloudFront
- Infrastructure teardown workflow
- Support for custom VPCs, security groups, and IAM roles
- Cross-account AWS access via OIDC
- Integration with Sembix Hub for shared services

## Prerequisites

Before using this template, ensure you have:

1. **AWS Account**: An AWS account where Sembix Studio will be deployed
2. **GitHub App**: A GitHub App configured for authentication (required for accessing private Terraform repositories)
3. **Terraform State Backend**: An S3 bucket for storing Terraform state
4. **ECR Repositories**: Container registries for Sembix Studio images
5. **Permissions**: Appropriate AWS IAM roles and permissions configured

## Getting Started

### 1. Use This Template

Click "Use this template" to create a new repository from this template.

### 2. Configure GitHub Environments

Set up GitHub environments (e.g., `dev`, `staging`, `prod`) with the required secrets and variables.

**Quick Setup with Backstage UI**: Visit [backstage.studio.sembix.ai](https://backstage.studio.sembix.ai) to quickly create and manage environment secrets and variables through a user-friendly interface.

### 3. Configure Required Secrets

Navigate to your repository's Settings > Secrets and variables > Actions, and configure the following secrets for each environment:

#### Authentication & Core AWS
- `APP_ID`: GitHub App ID for authentication
- `APP_PRIVATE_KEY`: GitHub App private key
- `AWS_REGION`: AWS region (e.g., `us-east-1`)
- `CUSTOMER_ROLE_ARN`: IAM role ARN for deployment
- `DEPLOYMENT_APPROVERS`: Comma-separated list of GitHub usernames who can approve deployments
- `IAM_ROLE_PATH`: (Optional) IAM role path for bootstrap (defaults to `/`)

#### ECR Repositories
- `ENGINE_ECR_REPOSITORY_URL`: ECR URL for workflow engine images
- `MIGRATIONS_ECR_REPOSITORY_URL`: ECR URL for migration images
- `BFF_ECR_REPOSITORY_URL`: ECR URL for BFF (Backend for Frontend) images

#### Sembix Hub Integration
- `SEMBIX_HUB_AWS_ACCOUNT_ID`: AWS account ID for Sembix Hub
- `SEMBIX_HUB_STUDIO_SECRET_NAME`: Secret name in Sembix Hub
- `SEMBIX_HUB_STUDIO_KMS_KEY_ID`: KMS key ID for encryption
- `SEMBIX_HUB_HUB_CONSUMER_ROLE_ARN`: Role ARN for Hub consumer access
- `SEMBIX_HUB_HUB_ADMIN_ROLE_ARN`: Role ARN for Hub admin access
- `SEMBIX_HUB_APP_CONFIG_ROLE_ARN`: Role ARN for AppConfig access
- `SEMBIX_HUB_APP_CONFIG_APP_ID`: AppConfig application ID
- `SEMBIX_HUB_APP_CONFIG_ENV_ID`: AppConfig environment ID
- `SEMBIX_HUB_APP_CONFIG_PROFILE_ID`: AppConfig configuration profile ID
- `SEMBIX_HUB_ENGINE_EXECUTION_ROLE`: Engine execution role ARN

#### Database Configuration
- `SEMBIX_STUDIO_DB_NAME`: Database name for Sembix Studio
- `SEMBIX_STUDIO_USER`: Database username

#### Storage
- `SEMBIX_STUDIO_ARTIFACT_S3_BUCKET`: S3 bucket for artifacts and storage

#### Optional: Custom Networking (if not using default VPC)
- `USE_CUSTOM_NETWORKING`: Set to `true` to use custom networking
- `CUSTOM_VPC_ID`: Your VPC ID
- `CUSTOM_PUBLIC_SUBNET_IDS`: Comma-separated public subnet IDs
- `CUSTOM_PRIVATE_SUBNET_IDS`: Comma-separated private subnet IDs
- `VPC_CIDR`: VPC CIDR block (if creating new VPC)
- `PUBLIC_SUBNET_CIDRS`: Public subnet CIDR blocks
- `PRIVATE_SUBNET_CIDRS`: Private subnet CIDR blocks
- `AZ_COUNT`: Number of availability zones
- `ENABLE_VPC_ENDPOINTS`: Enable VPC endpoints for AWS services

#### Optional: Custom Security Groups
- `USE_CUSTOM_SECURITY_GROUPS`: Set to `true` to use custom security groups
- `CUSTOM_WORKFLOW_ENGINE_SG_ID`: Security group for workflow engine
- `CUSTOM_AURORA_SG_ID`: Security group for Aurora database
- `CUSTOM_RDS_PROXY_SG_ID`: Security group for RDS Proxy
- `CUSTOM_BFF_ECS_SG_ID`: Security group for BFF ECS tasks
- `CUSTOM_BASTION_SG_ID`: Security group for bastion host
- `CUSTOM_BFF_ALB_SG_ID`: Security group for BFF load balancer

#### Optional: Custom IAM Roles
- `USE_CUSTOM_IAM_POLICIES`: Set to `true` to use custom IAM roles
- `WORKFLOW_ENGINE_EXECUTION_ROLE_ARN`: ECS execution role for workflow engine
- `WORKFLOW_ENGINE_TASK_ROLE_ARN`: ECS task role for workflow engine
- `BFF_ECS_TASK_EXECUTION_ROLE_ARN`: ECS execution role for BFF
- `BFF_ECS_TASK_ROLE_ARN`: ECS task role for BFF
- `RDS_PROXY_ROLE_ARN`: IAM role for RDS Proxy
- `SEMBIX_STUDIO_MEMORY_ROLE_ARN`: IAM role for memory service
- `SEMBIX_STUDIO_NOTIFICATION_ROLE_ARN`: IAM role for notification service

#### Optional: TLS/SSL & DNS
- `SEMBIX_STUDIO_CF_VIEWER_CERT`: CloudFront viewer certificate ARN
- `BFF_ALB_CERTIFICATE_ARN`: ACM certificate ARN for BFF ALB
- `HOSTED_ZONE_ID`: Route53 hosted zone ID

#### Optional: Bastion Host
- `BASTION_AMI_ID`: AMI ID for bastion host
- `BASTION_SSH_KEY_NAME`: SSH key pair name for bastion access

### 4. Configure Repository Variables

Set the following variables in Settings > Secrets and variables > Actions > Variables:

- `TF_BUCKET_NAME`: S3 bucket name for Terraform state
- `CLOUDFRONT_DOMAIN`: Domain name for CloudFront distribution
- `VITE_GITHUB_APP_CLIENT_ID`: GitHub App client ID for frontend
- `VITE_GITHUB_APP_NAME`: GitHub App name for frontend
- `VITE_JIRA_CLIENT_ID`: Jira client ID (if using Jira integration)
- `ENABLE_BPA`: Enable Business Process Automation module (true/false)
- `DEPLOY_SEMBIX_STUDIO_MEMORY`: Deploy memory service (true/false)
- `DEPLOY_SEMBIX_STUDIO_NOTIFICATIONS`: Deploy notification service (true/false)
- `ENABLE_BFF_WAF`: Enable WAF for BFF (true/false)

## Deploying Sembix Studio

### First-Time Deployment (Bootstrap)

For the first deployment to a new environment, you must run the bootstrap phase first:

#### Phase 1: Bootstrap

1. Go to **Actions** > **Deploy Sembix Studio**
2. Click **Run workflow**
3. Configure the bootstrap:
   - **version**: Sembix Studio version (e.g., `v1.2.3` or `main`)
   - **terraform_ref**: Terraform workflow version (default: `v3.0.0`)
   - **environment**: Target environment name
   - **deployment_phase**: Select **bootstrap**
   - **require_approval**: Select **true** for manual approval (recommended)
4. Click **Run workflow**
5. Save the IAM role ARNs from the bootstrap output - you'll need these for the deploy phase

#### Phase 2: Main Deployment

After bootstrap completes and offline communications are finalized:

1. Go to **Actions** > **Deploy Sembix Studio**
2. Click **Run workflow**
3. Configure the deployment:
   - **version**: Same Sembix Studio version used in bootstrap
   - **terraform_ref**: Same Terraform version (default: `v3.0.0`)
   - **environment**: Same environment name
   - **deployment_phase**: Select **deploy**
   - **run_migrations**: Select **true** to run database migrations
   - **deploy_ui**: Select **true** to deploy the UI
   - **require_approval**: Select **true** for safety (recommended)
4. Click **Run workflow**

### Subsequent Deployments

For updates to an existing environment (after initial bootstrap and deploy):

1. Go to **Actions** > **Deploy Sembix Studio**
2. Click **Run workflow**
3. Configure the deployment:
   - **version**: New version to deploy
   - **terraform_ref**: Terraform version (default: `v3.0.0`)
   - **environment**: Target environment
   - **deployment_phase**: Select **deploy** (bootstrap is only needed once)
   - **run_migrations**: Select **true** if database changes are included
   - **deploy_ui**: Select **true** to update the UI
   - **require_approval**: Select **true** for safety
4. Click **Run workflow**

## Workflow Details

### Deploy Workflow

The deployment workflow is split into two separate phases based on the `deployment_phase` input:

#### Bootstrap Phase

When `deployment_phase` is set to **bootstrap**:

1. **Validate Bootstrap Configuration**: Validates bootstrap-specific configuration
   - Verifies AWS authentication
   - Checks IAM role path settings
   - Validates terraform state bucket access
2. **Bootstrap Infrastructure**: Creates foundational IAM roles
   - Creates ECS task execution and task roles
   - Sets up RDS Proxy IAM role
   - Configures cross-account access policies
   - **Important**: Save the output IAM role ARNs for the deploy phase

#### Deploy Phase

When `deployment_phase` is set to **deploy**:

1. **Validate Deploy Configuration**: Validates deployment configuration
   - Verifies all ECR repository URLs
   - Checks Sembix Hub integration settings
   - Validates database, networking, and security configurations
   - Confirms IAM roles exist (from bootstrap)
2. **Deploy Sembix Studio**: Deploys the full Sembix Studio stack
   - Deploys infrastructure via Terraform
   - Sets up VPC, subnets, and security groups (if not using custom)
   - Creates Aurora PostgreSQL database with RDS Proxy
   - Deploys ECS services (workflow engine, BFF)
   - Runs database migrations (if enabled)
   - Deploys UI to S3/CloudFront (if enabled)
   - Configures Application Load Balancer and DNS

**Note**: Bootstrap and deploy are run as separate workflow executions. This allows for an offline communication pause between creating IAM roles and deploying the main infrastructure.

### Destroy Workflow

Use the destroy workflow to tear down an environment:

1. Go to **Actions** > **Destroy Sembix Studio**
2. Click **Run workflow**
3. Configure:
   - **version**: Version reference
   - **terraform_ref**: Terraform version used for deployment (default: `v3.0.0`)
   - **environment**: Environment to destroy
   - **require_approval**: Highly recommended to leave as **true**
4. Click **Run workflow**

**Warning**: This action is destructive and will delete all resources in the specified environment. Note that IAM roles created by the bootstrap phase will remain and must be destroyed separately if needed.

## Managing Configuration with Backstage UI

For a streamlined configuration management experience, Sembix provides a Backstage UI hosted at [backstage.studio.sembix.ai](https://backstage.studio.sembix.ai).

### Features

- **Quick Environment Setup**: Create GitHub environments with all required secrets and variables
- **Template-Based Configuration**: Use pre-configured templates for common deployment scenarios
- **Validation**: Real-time validation of configuration values
- **Secure**: Direct integration with GitHub's secrets API
- **Documentation**: Inline help for each configuration parameter

### Using Backstage UI

1. Navigate to [backstage.studio.sembix.ai](https://backstage.studio.sembix.ai)
2. Authenticate with your GitHub account
3. Select your repository from the deployment template
4. Choose your target environment (dev, staging, prod, etc.)
5. Fill in the configuration form with your environment-specific values
6. Review and submit to create all secrets and variables automatically

This eliminates the need to manually configure secrets in GitHub's UI, reducing setup time and potential configuration errors.

## Architecture

Sembix Studio deploys on AWS with the following components:

- **Compute**: ECS Fargate for containerized workloads
- **Database**: Aurora PostgreSQL with RDS Proxy
- **Storage**: S3 for artifacts and data
- **CDN**: CloudFront for UI distribution
- **Networking**: VPC with public/private subnets
- **Load Balancing**: Application Load Balancer for BFF
- **Security**: Security groups, IAM roles, and KMS encryption

## Support

For issues or questions:

- Review the [Sembix Studio Documentation](https://docs.sembix.ai)
- Check the GitHub Actions run logs for deployment errors
- Contact Sembix support at support@sembix.ai

## License

This template is provided by Sembix. See your Sembix Studio license agreement for terms and conditions.
