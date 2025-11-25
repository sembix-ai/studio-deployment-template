# Sembix Studio Deployment - First-Time Setup Guide

This guide walks you through the initial setup required to deploy Sembix Studio using this deployment template. Follow these steps in order for a successful first-time deployment.

## Table of Contents

1. [GitHub App Setup](#github-app-setup)
2. [Repository Configuration](#repository-configuration)
3. [Verification](#verification)

---

## GitHub App Setup

The deployment workflows require a GitHub App to authenticate and access the private `sembix-ai/sembix-studio-infrastructure` repository during deployment.

### Why is a GitHub App Required?

The GitHub App provides secure, scoped access to download Terraform modules and deployment files from private repositories. It uses read-only permissions for:
- Repository contents (to clone and download files)
- Repository metadata (to verify access)

**Important**: The GitHub App will be owned by your organization and needs to access repositories across multiple GitHub organizations (your organization and the Sembix organization).

### Creating the GitHub App

**Option 1: Using the GitHub App Manifest (Recommended)**

The `github-app-manifest.json` file in this repository contains the pre-configured settings. To use it:

1. Go to [GitHub Settings > Developer Settings > GitHub Apps](https://github.com/settings/apps)
2. Click **New GitHub App**
3. Look for the "Create a GitHub App from a manifest" link at the top of the form
4. Paste the contents of `github-app-manifest.json` from this repository
5. Click **Create GitHub App from manifest**

**Option 2: Manual Creation**

If the manifest approach doesn't work, follow the [Manual Setup](#alternative-manual-setup-if-needed) instructions below.

**After creating the app:**
1. Generate and download the private key (scroll to "Private keys" section and click "Generate a private key")
2. Note the App ID (shown at the top of the GitHub App settings page)
3. Install the app in your organization (click "Install App" in the sidebar)
4. Contact Sembix support at support@sembix.ai to request installation of the app in the `sembix-ai` organization - provide them with your App ID

### Alternative: Manual Setup (If Needed)

If you need to create the GitHub App manually:

1. Navigate to [GitHub Settings > Developer Settings > GitHub Apps](https://github.com/settings/apps) in your organization
2. Click **New GitHub App**
3. Fill in the form:
   - **GitHub App name**: `Sembix Studio Deploy` (or any unique name)
   - **Description**: `Provides read-only access so Sembix Studio deploy and destroy workflows can download Terraform modules and deployment files.`
   - **Homepage URL**: `https://sembix.ai`
   - **Webhook**: Uncheck "Active" (not needed)
   - **Permissions**:
     - Repository permissions > **Contents**: Read-only
     - Repository permissions > **Metadata**: Read-only (automatically set)
   - **Where can this GitHub App be installed?**: Select **"Any account"** (required for cross-organization access)
4. Click **Create GitHub App**
5. Generate and download the private key
6. Install the app in your organization with access to your deployment repository
7. Contact Sembix to install the app in the `sembix-ai` organization

### Configuring Repository Secrets

After creating the GitHub App, add the following values to your repository secrets (see [Repository Configuration](#repository-configuration) section below):

1. **`APP_ID`**: The numeric GitHub App ID (e.g., `123456`)
2. **`APP_PRIVATE_KEY`**: The complete private key in PEM format (including `-----BEGIN RSA PRIVATE KEY-----` and `-----END RSA PRIVATE KEY-----` lines)

### Security Notes

- The private key is sensitive and should be treated like a password
- Never commit the private key to your repository
- The GitHub App only has read-only access to repository contents
- You can revoke access at any time from the GitHub App settings

---

## Repository Configuration

Configure your deployment repository with the required secrets and variables.

### Setting up GitHub Secrets

Navigate to your repository: **Settings > Secrets and variables > Actions > Secrets > Repository secrets**

Configure the following repository-level secrets (these are shared across all environments):

**GitHub App Authentication**:
- `APP_ID`: The App ID from your GitHub App (e.g., `123456`)
- `APP_PRIVATE_KEY`: The entire contents of the `.pem` private key file you downloaded

**ECR Repositories** (provided by Sembix support):
- `ENGINE_ECR_REPOSITORY_URL`: Full ECR URL for workflow engine (e.g., `123456789012.dkr.ecr.us-east-1.amazonaws.com/sembix-studio-engine`)
- `MIGRATIONS_ECR_REPOSITORY_URL`: Full ECR URL for migrations
- `BFF_ECR_REPOSITORY_URL`: Full ECR URL for BFF (Backend for Frontend)
- `SEMANTIC_EVENT_ENGINE_ECR_REPOSITORY_URL`: Full ECR URL for semantic engine.

**Sembix Hub Integration** (provided by Sembix support):
- `SEMBIX_HUB_AWS_ACCOUNT_ID`: AWS account ID for Sembix Hub
- `SEMBIX_HUB_STUDIO_SECRET_NAME`: Secret name in Sembix Hub for this deployment
- `SEMBIX_HUB_STUDIO_KMS_KEY_ID`: KMS key ID for encrypting Studio secrets
- `SEMBIX_HUB_APP_CONFIG_APP_ID`: AppConfig application ID
- `SEMBIX_HUB_APP_CONFIG_ENV_ID`: AppConfig environment ID

**Storage** (provided by Sembix support):
- `SEMBIX_STUDIO_ARTIFACT_S3_BUCKET`: S3 bucket for artifacts and workflow data

**Note**: Contact Sembix support at support@sembix.ai to obtain the ECR repository URLs, Sembix Hub integration values, and artifact bucket name for your deployment.

---

## Verification

Before running your first deployment, verify your configuration:

### 1. Verify GitHub App

Check that:
- [ ] GitHub App is created and installed in your organization
- [ ] GitHub App has access to your deployment repository
- [ ] Sembix has been contacted to install the app in the `sembix-ai` organization
- [ ] `APP_ID` secret is set in repository secrets
- [ ] `APP_PRIVATE_KEY` secret is set in repository secrets with the full PEM contents

### 2. Test GitHub Actions Access

1. Go to **Actions** tab in your repository
2. Check that workflows are enabled
3. Verify that the "Deploy Sembix Studio" workflow appears

### 3. Configuration Checklist

Use this checklist to ensure all required repository-level configuration is complete:

**Repository Secrets** (Settings > Secrets and variables > Actions > Repository secrets):
- [ ] `APP_ID` - GitHub App ID
- [ ] `APP_PRIVATE_KEY` - GitHub App private key (full PEM contents)
- [ ] `ENGINE_ECR_REPOSITORY_URL` - ECR URL for engine (from Sembix support)
- [ ] `MIGRATIONS_ECR_REPOSITORY_URL` - ECR URL for migrations (from Sembix support)
- [ ] `BFF_ECR_REPOSITORY_URL` - ECR URL for BFF (from Sembix support)
- [ ] `SEMANTIC_EVENT_ENGINE_ECR_REPOSITORY_URL` - ECR URL for semantic engine (from Sembix support)
- [ ] `SEMBIX_HUB_AWS_ACCOUNT_ID` - Hub account ID (from Sembix support)
- [ ] `SEMBIX_HUB_STUDIO_SECRET_NAME` - Hub secret name (from Sembix support)
- [ ] `SEMBIX_HUB_STUDIO_KMS_KEY_ID` - Hub KMS key (from Sembix support)
- [ ] `SEMBIX_HUB_APP_CONFIG_APP_ID` - AppConfig app ID (from Sembix support)
- [ ] `SEMBIX_HUB_APP_CONFIG_ENV_ID` - AppConfig env ID (from Sembix support)
- [ ] `SEMBIX_STUDIO_ARTIFACT_S3_BUCKET` - Artifact S3 bucket (from Sembix support)


---

## Next Steps

Once repository setup is complete, proceed to the [README.md](README.md) for next steps including:
- Configuring GitHub environments via Backstage UI
- Deployment instructions

---

## Troubleshooting

### GitHub App Authentication Failures

**Symptom**: Workflows fail with "Failed to create GitHub App token" or similar errors.

**Solutions**:
- Verify `APP_ID` is a numeric value (not a string with quotes)
- Verify `APP_PRIVATE_KEY` includes the complete PEM file with header/footer lines
- Check that the GitHub App is installed in your organization
- Verify the GitHub App has been installed in the `sembix-ai` organization (contact Sembix support)
- Ensure the GitHub App has access to both your deployment repository and `sembix-ai/sembix-studio-infrastructure`

### Missing Repository Secrets

**Symptom**: Workflows fail with missing secret errors.

**Solutions**:
- Verify all 11 repository secrets are configured
- Check that secret names match exactly (case-sensitive)
- Ensure secrets are set at the repository level (not environment level)
- Contact Sembix support if you're missing values for ECR, Hub, or artifact bucket secrets

### Need Help?

Contact Sembix support at support@sembix.ai or refer to the [Sembix Studio Documentation](https://docs.sembix.ai).
