# GitHub Actions Workflow: Flow Terraform

This workflow template automates the process of planning, validating, and applying Terraform configurations in Azure using GitHub Actions. It is designed to be **reusable** via `workflow_call` and requires specific inputs and secrets.

---

## Workflow Overview

**Stages:**
1. **Plan**
   - Checks out code
   - Initializes Terraform backend using Azure Storage Account
   - Runs `terraform plan` with Azure credentials
   - Uploads the plan artifact
2. **Wait for Validation**
   - Pauses for manual approval before applying changes
3. **Apply**
   - Downloads the plan artifact
   - Initializes Terraform backend again
   - Applies the Terraform plan

---

## Inputs

| Name                  | Type   | Required | Description                                                      |
|-----------------------|--------|----------|------------------------------------------------------------------|
| `environment`         | string | Yes      | Environment name (e.g., dev, prod)                               |
| `appName`             | string | Yes      | Application name, used for backend container                     |
| `terraformPathFolder` | string | No       | Path to the Terraform configuration folder                       |
| `resoureGroupShared`  | string | No       | Name of the shared resource group for the backend                |
| `storageAccountShared`| string | No       | Name of the shared storage account for the backend               |

---

## Secrets

| Name                           | Required | Description                                              |
|--------------------------------|----------|----------------------------------------------------------|
| `AZURE_APPCONFIG_CONNECTION_STRING` | Yes  | Azure App Configuration connection string                |
| `AZURE_SAKEY`                  | Yes      | Azure Storage Account access key for backend             |
| `AZURE_CLIENT_ID`              | Yes      | Azure Service Principal Client ID                        |
| `AZURE_CLIENT_SECRET`          | Yes      | Azure Service Principal Client Secret                    |
| `AZURE_SUBSCRIPTION_ID`        | Yes      | Azure Subscription ID                                    |
| `AZURE_TENANT_ID`              | Yes      | Azure Tenant ID                                          |

---

## Steps Breakdown

### 1. Plan

- **Checkout code:**  
  Uses `actions/checkout@v4` to get the repository content.

- **Setup Terraform:**  
  Uses `hashicorp/setup-terraform@v3` to install Terraform.

- **Terraform Init:**  
  Initializes the backend with Azure Storage Account using provided secrets and inputs.

- **Terraform Plan:**  
  Runs `terraform plan` with Azure credentials as variables and outputs a plan file.

- **Upload Plan Artifact:**  
  Uploads the generated plan file as an artifact for later use.

### 2. Wait for Validation

- **Manual Approval:**  
  Uses `trstringer/manual-approval@v1` to require manual approval before applying changes.

### 3. Apply

- **Checkout code:**  
  Checks out the code again for context.

- **Download Plan Artifact:**  
  Downloads the previously uploaded plan artifact.

- **Setup Terraform:**  
  Installs Terraform again.

- **Terraform Init:**  
  Re-initializes the backend.

- **Terraform Apply:**  
  Applies the previously generated plan file.

---

## Example Usage

```yaml
jobs:
  call-terraform-flow:
    uses: your-org/your-repo/.github/workflows/terraform-flow.yml@main
    with:
      environment: 'dev'
      appName: 'my-app'
      terraformPathFolder: './infra'
      resoureGroupShared: 'my-shared-rg'
      storageAccountShared: 'mystorageaccount'
    secrets:
      AZURE_APPCONFIG_CONNECTION_STRING: ${{ secrets.AZURE_APPCONFIG_CONNECTION_STRING }}
      AZURE_SAKEY: ${{ secrets.AZURE_SAKEY }}
      AZURE_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
      AZURE_CLIENT_SECRET: ${{ secrets.AZURE_CLIENT_SECRET }}
      AZURE_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      AZURE_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
```

## Requirements

- The Azure Service Principal must have sufficient permissions to manage resources defined in your Terraform code.
- The Storage Account and Resource Group for the backend must exist and be accessible.
- The Terraform configuration must support the variables passed in the workflow.
- Service Principal must have configured the **Contributer** role in the Subscription ( and User **Access Administrator** if it needed to give permissons to users (like in hackaton-ai)).

## Troubleshooting

- **Backend errors:**
   Ensure the storage account, container, and resource group exist and the access key is correct. 
- **Authentication errors:**
   Double-check the Azure Service Principal credentials and permissions.
- **Manual approval:**
   The workflow will pause until the specified approver approves the plan.