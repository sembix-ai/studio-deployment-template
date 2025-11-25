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

## Getting Started

### 1. First-Time Setup

**New to this repository?** Start with the [Setup Guide (SETUP.md)](SETUP.md) to configure your repository with the required GitHub App and secrets.

The setup guide will walk you through:
- Creating a GitHub App for authentication
- Configuring repository-level secrets
- Obtaining necessary values from Sembix support

### 2. Use This Template

Click "Use this template" to create a new repository from this template.

### 3. Create Environment for Bootstrap

Use the **Backstage UI** at [backstage.studio.sembix.ai](https://backstage.studio.sembix.ai) to create your first GitHub environment.

1. Navigate to [backstage.studio.sembix.ai](https://backstage.studio.sembix.ai)
2. Authenticate with your GitHub account
3. Select your deployment repository
4. Run **Step 1: Setup New Sembix Studio Client Environment**
   - Choose your environment name (e.g., `dev`, `staging`, `prod`)
   - Fill in the initial configuration values
   - This creates a GitHub environment ready for bootstrapping

**You are now ready to bootstrap your environment!** See [Deploying Sembix Studio](#deploying-sembix-studio) below.

## Deploying Sembix Studio

### First-Time Deployment

First-time deployments require multiple phases with coordination between you and Sembix support.

#### Phase 1: Bootstrap Infrastructure

After completing **Step 1: Setup New Sembix Studio Client Environment** in Backstage UI:

1. Go to **Actions** > **Deploy Sembix Studio**
2. Click **Run workflow**
3. Configure the bootstrap:
   - **version**: Sembix Studio version (e.g., `v1.2.3`)
   - **terraform_ref**: Terraform workflow version (default: `v3.0.0`)
   - **environment**: Target environment name (must match what you created in Backstage)
   - **deployment_phase**: Select **bootstrap**
   - **require_approval**: Select **true** for manual approval (recommended)
4. Click **Run workflow**
5. **Save the IAM role ARNs from the bootstrap output**

#### Phase 2: Sembix Hub Integration Setup

After bootstrap completes:

1. **Send the IAM role ARNs** from the bootstrap output to Sembix support at support@sembix.ai
2. Sembix support will configure Sembix Hub integration for your deployment
3. Once Sembix support confirms the Hub configuration is complete, proceed to the next step

#### Phase 3: Add Hub Integration to Environment

After Sembix support confirms Hub setup:

1. Navigate to [backstage.studio.sembix.ai](https://backstage.studio.sembix.ai)
2. Select your deployment repository and environment
3. Run **Step 2: Add Sembix Studio Hub Integration**
   - This updates your GitHub environment with the Hub integration secrets and variables
   - These values enable communication between your deployment and Sembix Hub

#### Phase 4: Main Deployment

After completing **Step 2: Add Sembix Studio Hub Integration** in Backstage UI:

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

**Your Sembix Studio environment is now deployed!**

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
