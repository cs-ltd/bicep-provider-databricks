# Databricks ARM Custom Resource Provider Integration with Bicep

This folder contains comprehensive examples showing how to integrate Azure Databricks infrastructure automation with Bicep using ARM Custom Resource Providers as an extensibility mechanism.

## Overview

Since custom Bicep extensibility providers are not yet available for general development, this solution uses ARM Custom Resource Providers to extend Azure's REST API with custom Databricks resources and actions, enabling declarative Databricks infrastructure management through Bicep templates.

## Architecture

```
┌─────────────────┐    ┌──────────────────────┐    ┌─────────────────────┐
│   Bicep Files   │───▶│  ARM Custom Resource │───▶│  Azure Functions    │
│                 │    │      Provider        │    │  (Databricks APIs)  │
└─────────────────┘    └──────────────────────┘    └─────────────────────┘
                                                              │
                                                              ▼
                                                    ┌─────────────────────┐
                                                    │ Databricks Workspace│
                                                    │   (Clusters, Jobs)  │
                                                    └─────────────────────┘
```

## Files Description

### Core Bicep Templates

- **`databricks-provider.bicep`** - Defines the ARM Custom Resource Provider with Azure Functions backend for handling Databricks API calls
- **`databricks-infrastructure.bicep`** - Creates Databricks resources (clusters, jobs, instance pools) using the custom provider
- **`databricks-operations.bicep`** - Handles Databricks operations (start/stop clusters, run jobs) via deployment scripts
- **`main.bicep`** - Main deployment template that orchestrates the complete infrastructure deployment

### CI/CD Integration

- **`azure-pipelines.yml`** - Azure DevOps pipeline for automated validation, staging, and production deployments

## Key Features

### ✅ **Complete Databricks Integration**
- **Cluster Management**: Create, start, stop, restart clusters with full configuration
- **Job Orchestration**: Define and trigger ETL/ML jobs with dependencies
- **Resource Optimization**: Instance pools for cost management
- **Security**: Data security modes, encryption, and access controls

### ✅ **Bicep Native Experience**
- **Declarative Infrastructure**: Define Databricks resources using Bicep syntax
- **Parameter Management**: Environment-specific configurations
- **Dependency Management**: Proper resource ordering and dependencies
- **Output Handling**: Resource IDs and endpoints for further automation

### ✅ **Azure DevOps Integration**
- **CI/CD Pipelines**: Automated validation and deployment
- **Environment Promotion**: Staging to production workflows
- **Secret Management**: Secure handling of Databricks tokens
- **Testing Integration**: Automated job execution and validation

## Usage Examples

### 1. Deploy Custom Provider

```bash
az deployment group create \
  --resource-group rg-databricks \
  --template-file databricks-provider.bicep \
  --parameters databricksWorkspaceUrl="https://adb-123456789.azuredatabricks.net"
```

### 2. Deploy Databricks Infrastructure

```bash
az deployment group create \
  --resource-group rg-databricks \
  --template-file main.bicep \
  --parameters environment=production \
               databricksWorkspaceUrl="https://adb-123456789.azuredatabricks.net"
```

### 3. Perform Cluster Operations

```bash
az deployment group create \
  --resource-group rg-databricks \
  --template-file databricks-operations.bicep \
  --parameters databricksProviderName="databricks-infrastructure-provider" \
               clusterId="0123-456789-abc123" \
               operationType="start"
```

## Custom Resource Types Supported

### Clusters
- **Resource Type**: `Microsoft.CustomProviders/resourceProviders/clusters`
- **Operations**: Create, update, delete, start, stop, restart
- **Features**: Auto-scaling, spot instances, security modes, init scripts

### Jobs
- **Resource Type**: `Microsoft.CustomProviders/resourceProviders/jobs`
- **Operations**: Create, update, delete, run, monitor
- **Features**: Multi-task workflows, scheduling, notifications, retries

### Instance Pools
- **Resource Type**: `Microsoft.CustomProviders/resourceProviders/instancePools`
- **Operations**: Create, update, delete
- **Features**: Cost optimization, auto-termination, disk configuration

### Notebooks
- **Resource Type**: `Microsoft.CustomProviders/resourceProviders/notebooks`
- **Operations**: Create, update, delete, import, export
- **Features**: Version control, parameterization, execution

## Custom Actions Supported

- **`startCluster`** - Start a Databricks cluster
- **`stopCluster`** - Stop a Databricks cluster
- **`restartCluster`** - Restart a Databricks cluster
- **`runJob`** - Trigger a Databricks job run
- **`getClusterStatus`** - Get cluster status information

## Prerequisites

1. **Azure Subscription** with appropriate permissions
2. **Databricks Workspace** already deployed
3. **Databricks Personal Access Token** stored in Azure Key Vault
4. **Azure Functions** runtime for the custom provider backend
5. **Azure DevOps** project for CI/CD (optional)

## Implementation Steps

1. **Deploy the Custom Provider**: Use `databricks-provider.bicep` to create the ARM Custom Resource Provider and Azure Functions backend
2. **Configure Authentication**: Set up Databricks personal access tokens in Azure Key Vault
3. **Deploy Infrastructure**: Use `databricks-infrastructure.bicep` to create clusters, jobs, and instance pools
4. **Automate Operations**: Use `databricks-operations.bicep` for cluster lifecycle management
5. **Set up CI/CD**: Implement the Azure DevOps pipeline for automated deployments

## Security Considerations

- **Authentication**: Uses Azure managed identity and Key Vault for secure token storage
- **Network Security**: HTTPS-only communication with Databricks APIs
- **Access Control**: Role-based access control for custom provider operations
- **Encryption**: TLS encryption for all API communications

## Cost Optimization

- **Spot Instances**: Configurable spot instance usage for cost savings
- **Auto-termination**: Automatic cluster termination to prevent idle costs
- **Instance Pools**: Shared instance pools for faster cluster startup and cost efficiency
- **Right-sizing**: Configurable node types and worker counts per environment

## Monitoring and Troubleshooting

- **Deployment Scripts**: Built-in error handling and status monitoring
- **Azure Functions Logs**: Detailed logging for custom provider operations
- **Databricks Audit Logs**: Integration with Databricks workspace audit logs
- **Azure Monitor**: Integration with Azure Monitor for alerting and dashboards

## Limitations

- **Custom Provider Preview**: ARM Custom Resource Providers are in preview
- **Function App Dependencies**: Requires Azure Functions for backend implementation
- **API Rate Limits**: Subject to Databricks API rate limiting
- **Regional Availability**: Limited by Azure Functions and Databricks regional availability

## Contributing

This is an example implementation demonstrating the feasibility of Databricks integration with Bicep through ARM Custom Resource Providers. For production use, consider:

1. Implementing comprehensive error handling in Azure Functions
2. Adding support for additional Databricks resources (Unity Catalog, SQL warehouses, etc.)
3. Implementing proper authentication and authorization mechanisms
4. Adding comprehensive monitoring and alerting
5. Implementing proper backup and disaster recovery procedures

## Related Resources

- [Azure Custom Resource Providers Documentation](https://docs.microsoft.com/en-us/azure/azure-resource-manager/custom-providers/)
- [Azure Databricks REST API Reference](https://docs.databricks.com/api/azure/workspace/introduction)
- [Bicep Documentation](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/)
- [Azure Functions Documentation](https://docs.microsoft.com/en-us/azure/azure-functions/)
