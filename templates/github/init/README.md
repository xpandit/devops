# Azure DevOps Pipeline Template: Initialize and Versioning

This template is designed to initialize the environment, fetch configuration variables from Azure App Configuration, and manage versioning using Azure Table Storage. It is intended to be used in Azure DevOps pipelines and follows Azure best practices.

---

## Parameters

The template accepts the following parameters:

| Parameter Name   | Type   | Default Value | Description                                                                 |
|-------------------|--------|---------------|-----------------------------------------------------------------------------|
| `appName`         | string | `''`          | The name of the application.                                               |
| `azureConnection` | string | `''`          | The name of the Azure service connection.                                  |
| `appConfigName`   | string | `''`          | The name of the Azure App Configuration instance.                          |
| `updateVersion`   | string | `1.0.1`       | The default version to use for versioning.                                 |

---

## Steps

### Step 1: Initialize the Environment and Load Variables

This step uses the `AzureAppConfiguration@8` task to fetch configuration variables from Azure App Configuration.

#### Inputs:
- **`azureSubscription`**: The Azure service connection name.
- **`configstoreName`**: The name of the App Configuration instance.
- **`AppConfigurationEndpoint`**: The endpoint URL for the App Configuration instance.
- **`keyFilter`**: Filters the keys to fetch (default: `global:shared:*`).
- **`labelFilter`**: Filters the labels to fetch (default: empty).
- **`TrimKeyPrefix`**: Trims the prefix from the keys (default: `global:shared:`).
- **`selectKeys`**: Indicates whether to select specific keys (default: `true`).

---

### Step 2: Versioning Task

This step uses a PowerShell script to manage versioning in Azure Table Storage. It performs the following actions:
1. Splits the `updateVersion` parameter into `major`, `minor`, and `patch` components.
2. Queries Azure Table Storage to check if an entry already exists for the given `appName`, `branchName`, and `version`.
3. If an entry exists:
   - Increments the `revision` if the `version` matches.
   - Updates the `version` and resets the `revision` to `1` if the `version` does not match.
4. If no entry exists:
   - Creates a new entry with the specified `version` and `revision=1`.
5. Updates the build number with the generated version.

#### Key Variables:
- **`$connectionString`**: The connection string for Azure Table Storage.
- **`$tableName`**: The name of the Azure Table Storage table (default: `VersioningTable`).
- **`$rowKey`**: Combines the `branch name` and `version` to create a unique key.

---

## Outputs

- **Generated Version**: The final version is displayed in the pipeline logs and set as the build number using `##vso[build.updatebuildnumber]`.

---

## Example Usage

```yaml
- template: [init-pipeline.yml](http://_vscodecontentref_/0)
  parameters:
    appName: 'my-app'
    azureConnection: 'my-azure-connection'
    appConfigName: 'my-app-config'
    updateVersion: '1.0.0'
```

## Prerequisites

1. `Azure App Configuration`:
   - Ensure the App Configuration instance is set up and contains the required keys and labels.
2. `Azure Table Storage`:
   - Ensure the Table Storage instance is set up with a table named `VersioningTable`.
3. `Azure Service Connection`:
   - Create a service connection in Azure DevOps with access to the required Azure resources.

---

## Notes

* The `updateVersion` parameter should follow the format `major.minor.patch` (e.g., `1.0.1`).
* The `RowKey` in Azure Table Storage is constructed as branchName-version to ensure uniqueness for different versions.

---

## Troubleshooting
Common Errors

1. `The specified entity already exists`:
   - This occurs if an entry with the same `PartitionKey` and `RowKey` already exists in the table. Ensure the `RowKey` includes the version to differentiate entries.
2. `ResourceNotFound`:
   - This occurs if the specified table or entity does not exist. Verify the `connectionString` and `tableName`.

---

## References

- [Azure App Configuration Documentation](https://learn.microsoft.com/en-us/azure/azure-app-configuration/)
- [Azure Table Storage Documentation](https://learn.microsoft.com/en-us/azure/storage/tables/)
- [Azure DevOps Pipeline Tasks](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/)