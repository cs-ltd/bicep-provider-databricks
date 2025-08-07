# GitHub Actions CI/CD Setup for Bicep Tests

This document describes how to set up the GitHub Actions workflow for running Bicep tests against a Databricks testing instance.

## Overview

The GitHub Actions workflow runs PowerShell Pester tests from the `bicep/tests/` directory against a Databricks testing instance. This provides automated testing for all Bicep modules without requiring complex multi-environment deployments.

## Prerequisites

### 1. Databricks Testing Instance

You need a Databricks workspace for testing. This can be:
- A dedicated testing workspace in Azure
- A development workspace with appropriate permissions
- Any Databricks workspace where you can create/delete resources for testing

### 2. Databricks Personal Access Token

Create a Personal Access Token (PAT) in your Databricks workspace:
1. Log into your Databricks workspace
2. Go to User Settings → Developer → Access tokens
3. Generate a new token with appropriate permissions
4. Save the token securely (you'll need it for GitHub secrets)

### 3. Azure Resource Group

The tests require an Azure resource group named `rg-bicep-databricks-test` in the `East US` region. Create this resource group in your Azure subscription:

```bash
az group create --name rg-bicep-databricks-test --location "East US"
```

## GitHub Repository Configuration

### Required Secrets

Add the following secrets to your GitHub repository (Settings → Secrets and variables → Actions):

| Secret Name | Description | Example Value |
|-------------|-------------|---------------|
| `DATABRICKS_WORKSPACE_URL` | Your Databricks workspace URL | `https://adb-123456789.azuredatabricks.net` |
| `DATABRICKS_TOKEN` | Databricks Personal Access Token | `dapi1234567890abcdef...` |

### Workflow Triggers

The workflow runs automatically on:
- **Pull Requests**: When changes are made to files in the `bicep/` directory
- **Push to main**: When changes to `bicep/` directory are merged to main branch
- **Manual trigger**: Using the "Run workflow" button in GitHub Actions

## Testing Strategy

### What Gets Tested

The workflow runs all PowerShell Pester tests in `bicep/tests/`:
- Individual Bicep module functionality
- Positive path tests (successful resource creation)
- Negative path tests (error handling and validation)
- Resource cleanup after test completion

### Test Environment

- **Runner**: Ubuntu latest
- **PowerShell**: Latest Azure PowerShell module
- **Test Framework**: Pester (PowerShell testing framework)
- **Resource Scope**: Tests create temporary resources in your Databricks workspace
- **Cleanup**: Tests automatically clean up created resources

### Test Execution Flow

1. Checkout repository code
2. Setup PowerShell and install Pester module
3. Run all tests in `bicep/tests/` directory with environment variables
4. Tests use `bicep/helpers/Invoke-DatabricksApi.ps1` for Databricks API calls
5. Automatic cleanup of test resources

## Troubleshooting

### Common Issues

**Authentication Failures**
- Verify `DATABRICKS_WORKSPACE_URL` is correct (include https://)
- Ensure `DATABRICKS_TOKEN` is valid and has appropriate permissions
- Check that the Databricks workspace is accessible

**Resource Group Errors**
- Confirm `rg-bicep-databricks-test` resource group exists in `East US`
- Verify you have Contributor permissions on the resource group

**Test Failures**
- Check individual test logs in GitHub Actions
- Verify Databricks workspace has required features enabled
- Ensure no resource conflicts from previous test runs

### Manual Testing

You can trigger the workflow manually to test the setup:
1. Go to Actions tab in GitHub repository
2. Select "Bicep Tests" workflow
3. Click "Run workflow" button
4. Monitor the execution and check logs

## Security Considerations

- Personal Access Tokens should be rotated regularly
- Use dedicated testing workspace separate from production
- Limit token permissions to minimum required for testing
- Monitor test resource usage to avoid unexpected costs

## Integration with Existing Workflows

This workflow complements the existing GitHub Actions:
- Runs alongside Go provider tests (`make test`)
- Follows same patterns as existing workflows
- Uses similar runner configuration and setup steps
- Integrates with existing PR review process
