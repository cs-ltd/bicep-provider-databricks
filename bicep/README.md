# Databricks Bicep Provider

This directory contains Bicep modules for deploying and managing Azure Databricks resources using direct REST API calls through a centralized helper.

## Overview

The Databricks Bicep Provider enables Infrastructure as Code (IaC) for Databricks resources through:
- Individual Bicep modules for each Databricks resource type
- Centralized `Invoke-DatabricksApi` helper for consistent API interactions
- Comprehensive test coverage using Pester framework
- Complete workspace deployment examples

## Architecture

The provider uses Azure Deployment Scripts to execute PowerShell commands that interact directly with the Databricks REST API:

1. **Centralized Helper**: `helpers/Invoke-DatabricksApi.ps1` handles all API calls with consistent authentication and error handling
2. **Bicep Modules**: Individual modules under `modules/` for each Databricks resource type
3. **Deployment Scripts**: Azure Deployment Scripts execute PowerShell to create/manage resources
4. **Testing Framework**: Pester tests in `tests/` directory for validation

## Available Modules

### Core Infrastructure
- `cluster.bicep` - Databricks clusters with auto-scaling and custom configurations
- `instance-pool.bicep` - Instance pools for cost-effective compute resource management
- `job.bicep` - Databricks jobs with scheduling and notification support

### Planned Modules (110+ total)
- `workspace.bicep` - Workspace configuration and settings
- `secret-scope.bicep` - Secret management and Key Vault integration
- `notebook.bicep` - Notebook deployment and management
- `library.bicep` - Library installation and management
- `policy.bicep` - Cluster policies and governance
- `permissions.bicep` - Access control and permissions
- And 100+ more modules covering all Terraform provider resources

## Usage

### 1. Basic Cluster Deployment

```bicep
module cluster 'modules/cluster.bicep' = {
  name: 'my-databricks-cluster'
  params: {
    ClusterName: 'production-cluster'
    SparkVersion: '13.3.x-scala2.12'
    NodeTypeId: 'Standard_DS3_v2'
    NumWorkers: 4
    DatabricksToken: databricksToken
    WorkspaceUrl: 'https://adb-123456789.azuredatabricks.net'
    CustomTags: {
      Environment: 'Production'
      Project: 'DataPlatform'
    }
  }
}
```

### 2. Auto-scaling Cluster with Instance Pool

```bicep
module instancePool 'modules/instance-pool.bicep' = {
  name: 'shared-instance-pool'
  params: {
    InstancePoolName: 'shared-compute-pool'
    NodeTypeId: 'Standard_DS3_v2'
    MinIdleInstances: 2
    MaxCapacity: 20
    DatabricksToken: databricksToken
    WorkspaceUrl: workspaceUrl
  }
}

module cluster 'modules/cluster.bicep' = {
  name: 'autoscaling-cluster'
  dependsOn: [instancePool]
  params: {
    ClusterName: 'auto-scaling-cluster'
    AutoScale: true
    MinWorkers: 2
    MaxWorkers: 10
    InstancePoolId: instancePool.outputs.InstancePoolId
    DatabricksToken: databricksToken
    WorkspaceUrl: workspaceUrl
  }
}
```

### 3. Scheduled ETL Job

```bicep
module etlJob 'modules/job.bicep' = {
  name: 'daily-etl-job'
  params: {
    JobName: 'daily-data-processing'
    DatabricksToken: databricksToken
    WorkspaceUrl: workspaceUrl
    JobSettings: {
      notebook_task: {
        notebook_path: '/Shared/ETL/daily-processing'
        base_parameters: {
          environment: 'production'
          date: '{{ds}}'
        }
      }
      existing_cluster_id: cluster.outputs.ClusterId
    }
    Schedule: {
      quartz_cron_expression: '0 0 2 * * ?'
      timezone_id: 'UTC'
    }
    EmailNotifications: {
      on_success: ['data-team@company.com']
      on_failure: ['ops-team@company.com']
    }
  }
}
```

### 4. Complete Workspace Deployment

See the comprehensive example in `examples/full-workspace-deploy/` which demonstrates:
- Shared instance pool for cost optimization
- Production and development clusters
- Multiple job types (ETL, ML training, data validation)
- Proper dependency management and resource tagging

## Authentication

### Databricks Personal Access Token

Store your Databricks PAT securely in Azure Key Vault:

```bash
az keyvault secret set \
  --vault-name "your-keyvault" \
  --name "databricks-token" \
  --value "your-databricks-pat"
```

Reference it in your Bicep templates:

```bicep
param databricksToken string = keyVault.getSecret('databricks-token')
```

### Required Permissions

Your Databricks token needs the following permissions:
- Cluster management (create, read, update, delete)
- Job management (create, read, update, delete, run)
- Instance pool management
- Workspace object access (notebooks, libraries)

## Testing

### Running Tests

Tests use the Pester framework and require environment variables:

```powershell
# Set required environment variables
$env:DATABRICKS_WORKSPACE_URL = "https://adb-123456789.azuredatabricks.net"
$env:DATABRICKS_TOKEN = "your-databricks-pat"

# Run all tests
Invoke-Pester -Path "tests/" -Recurse

# Run specific module tests
Invoke-Pester -Path "tests/cluster.Tests.ps1"
```

### Test Structure

Each module has corresponding tests that cover:
- **Positive Path**: Successful resource creation with various configurations
- **Negative Path**: Validation of error handling and parameter constraints
- **Cleanup**: Automatic resource cleanup after test completion

## Development Guidelines

### Adding New Modules

When adding a new Databricks resource module:

1. **Research the Terraform Resource**: Examine the corresponding Terraform resource in `/internal/.../*.go`
2. **Map Parameters**: Convert Terraform schema to Bicep parameters with proper types and validation
3. **Identify REST API**: Determine the Databricks REST API endpoints and methods
4. **Create Module**: Implement the Bicep module using the established patterns
5. **Add Tests**: Create comprehensive Pester tests for the module
6. **Update Documentation**: Add usage examples and parameter descriptions

### Coding Standards

- **Parameter Naming**: Use PascalCase for all parameters and outputs
- **Descriptions**: Every parameter and output must have an `@description` annotation
- **Security**: Use `@secure()` for sensitive parameters like tokens and passwords
- **Validation**: Add `@allowed()` constraints where appropriate
- **Dependencies**: Use `dependsOn` for proper resource ordering
- **Error Handling**: Include comprehensive error handling in deployment scripts

### Module Structure Template

```bicep
@description('Description of the main resource identifier')
param ResourceName string

@description('Databricks Personal Access Token')
@secure()
param DatabricksToken string

@description('Databricks workspace URL')
param WorkspaceUrl string

// Additional parameters with proper types and validation

var resourceConfig = {
  // Map parameters to API payload structure
}

resource resourceCreation 'Microsoft.Resources/deploymentScripts@2023-08-01' = {
  name: 'create-databricks-resource-${uniqueString(ResourceName)}'
  location: resourceGroup().location
  kind: 'AzurePowerShell'
  properties: {
    azPowerShellVersion: '9.0'
    timeout: 'PT30M'
    retentionInterval: 'PT1H'
    environmentVariables: [
      {
        name: 'DATABRICKS_TOKEN'
        secureValue: DatabricksToken
      }
      {
        name: 'WORKSPACE_URL'
        value: WorkspaceUrl
      }
      {
        name: 'RESOURCE_CONFIG'
        value: string(resourceConfig)
      }
    ]
    scriptContent: '''
      # PowerShell script using Invoke-DatabricksApi helper
    '''
  }
}

@description('Output description')
output ResourceId string = resourceCreation.properties.outputs.resourceId
```

## Migration from ARM Custom Resource Provider

If you're migrating from the previous ARM Custom Resource Provider approach:

1. **Replace Provider References**: Remove custom provider dependencies
2. **Update Resource Types**: Change from custom resource types to deployment scripts
3. **Modify Authentication**: Update to use direct token passing instead of provider authentication
4. **Test Thoroughly**: Validate all functionality works with the new direct API approach

## Best Practices

### Security
- Always use Azure Key Vault for storing Databricks tokens
- Never hardcode sensitive values in templates
- Use managed identities where possible for Azure resource access
- Implement proper RBAC for deployment permissions

### Resource Management
- Use consistent naming conventions across all resources
- Apply comprehensive tagging for cost tracking and governance
- Implement proper lifecycle management with auto-termination settings
- Monitor resource utilization and optimize configurations

### Performance
- Use instance pools for cost-effective compute resource sharing
- Configure auto-scaling based on actual workload patterns
- Enable Photon runtime for improved performance where applicable
- Optimize Spark configurations for your specific use cases

### Monitoring and Alerting
- Set up email notifications for job failures
- Monitor cluster utilization and costs
- Implement alerting for long-running or failed deployments
- Use Azure Monitor for deployment script execution tracking

## Troubleshooting

### Common Issues

1. **Authentication Failures**
   ```
   Error: Databricks API call failed: Unauthorized (Status: 401)
   ```
   - Verify Databricks token is valid and not expired
   - Check token permissions for the required operations
   - Ensure workspace URL is correct and accessible

2. **Resource Creation Timeouts**
   ```
   Error: Cluster creation timed out after 30 attempts
   ```
   - Check Databricks workspace capacity and quotas
   - Verify node types are available in the workspace region
   - Review cluster configuration for invalid settings

3. **Parameter Validation Errors**
   ```
   Error: The template parameter 'NodeTypeId' is not valid
   ```
   - Verify parameter values match Databricks API requirements
   - Check for typos in node type IDs or other identifiers
   - Ensure required parameters are provided

### Debugging Deployment Scripts

1. **View Script Logs**: Check the deployment script execution logs in Azure portal
2. **Enable Verbose Logging**: Add `Write-Host` statements for debugging
3. **Test API Calls**: Use the `Invoke-DatabricksApi` helper directly for testing
4. **Validate JSON**: Ensure configuration objects are properly formatted

### Getting Help

- **Databricks API Documentation**: https://docs.databricks.com/dev-tools/api/latest/
- **Azure Deployment Scripts**: https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/deployment-script-template
- **Bicep Documentation**: https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/

## Contributing

Contributions are welcome! Please:

1. Follow the established coding standards and patterns
2. Include comprehensive tests for new modules
3. Update documentation with usage examples
4. Ensure all CI checks pass before submitting PRs
5. Keep PRs focused and under 600 lines of changes

## Roadmap

### Phase 1 (Current)
- ✅ Core infrastructure modules (cluster, instance-pool, job)
- ✅ Centralized API helper
- ✅ Testing framework
- ✅ Complete workspace deployment example

### Phase 2 (Next)
- 🔄 Workspace configuration modules
- 🔄 Security and permissions modules
- 🔄 Data source and mount point modules
- 🔄 Advanced job types and workflows

### Phase 3 (Future)
- ⏳ All 110+ Terraform resource equivalents
- ⏳ Advanced features (Unity Catalog, MLflow)
- ⏳ CI/CD integration examples
- ⏳ Multi-workspace deployment patterns

## License

This project is licensed under the MIT License - see the LICENSE file for details.
