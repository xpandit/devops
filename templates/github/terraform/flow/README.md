# GitHub Actions Terraform Workflow

This project provides a GitHub Actions workflow for managing Terraform deployments. The workflow is defined in the `.github/workflows/terraform.yml` file and includes jobs for planning and applying Terraform configurations, as well as waiting for manual validation.

## Workflow Overview

The workflow consists of the following key jobs:

1. **PlanTerraform**: This job initializes Terraform and creates an execution plan. It uses the specified Azure connection and configuration parameters.

2. **WaitForValidation**: After planning, this job pauses for manual validation. Users are notified to review the configuration and can resume the workflow upon approval.

3. **ApplyTerraform**: Once validation is complete, this job applies the Terraform configuration to provision the resources in Azure.

## Prerequisites

- Ensure you have a valid Azure account and the necessary permissions to create resources.
- Set up the required secrets in your GitHub repository for Azure authentication.

## Configuration

To use this workflow, you need to define the following parameters in your GitHub repository secrets:

- `AZURE_CONNECTION`: Your Azure service connection details.
- `APP_NAME`: The name of your application.
- `ENVIRONMENT`: The environment for deployment (e.g., dev, prod).
- `TERRAFORM_PATH_FOLDER`: The path to your Terraform configuration files.
- `RESOURCE_GROUP_SHARED`: The name of the shared resource group.
- `STORAGE_ACCOUNT_SHARED`: The name of the shared storage account.
- `APP_CONFIG_NAME`: The name of your Azure App Configuration.

## Usage

1. Clone this repository to your local machine.
2. Modify the workflow file as needed to fit your project requirements.
3. Push your changes to the GitHub repository.
4. The workflow will trigger on specified events (e.g., push, pull request) and execute the defined jobs.

## License

This project is licensed under the MIT License. See the LICENSE file for more details.