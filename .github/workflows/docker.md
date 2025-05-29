# GitHub Actions Workflow: Flow Docker

This workflow template automates building, pushing, and deploying Docker images to Azure Container Apps using Azure Container Registry (ACR). It is designed to be **reusable** via `workflow_call` and requires specific inputs and secrets.

---

## Workflow Overview

**Stages:**
1. **Build and Publish**
   - Checks out code
   - Generates a `.env` file from a template
   - Authenticates with Azure and ACR
   - Builds and pushes a Docker image to ACR
2. **Wait for Validation**
   - Pauses for manual approval before deployment
3. **Deploy Docker**
   - Deploys the pushed image to Azure Container Apps

---

## Inputs

| Name                | Type   | Required | Description                                   |
|---------------------|--------|----------|-----------------------------------------------|
| `vmImageName`       | string | Yes      | VM image for the runner (e.g., `ubuntu-latest`) |
| `resourceGroupName` | string | Yes      | Azure Resource Group name                     |
| `containerRegistry` | string | Yes      | Azure Container Registry login server (e.g., `myregistry.azurecr.io`) |
| `containerAppName`  | string | Yes      | Azure Container App name                      |
| `dockerfilePath`    | string | Yes      | Path to the Dockerfile (e.g., `./Dockerfile`) |
| `imageRepository`   | string | Yes      | Name of the image repository in ACR           |
| `tag`               | string | Yes      | Docker image tag (e.g., `latest`)             |

---

## Secrets

| Name                | Required | Description                                      |
|---------------------|----------|--------------------------------------------------|
| `AZURE_CREDENTIALS` | Yes      | Azure service principal credentials as JSON. Must have permissions to push to ACR and deploy to Container Apps. |

---

## Steps Breakdown

### 1. Build and Publish

- **Checkout code:**  
  Uses `actions/checkout@v4` to get the repository content.

- **Create .env file:**  
  Reads `.env.template` and creates a `.env` file, filling in environment variables if available.

- **Azure Login:**  
  Authenticates to Azure using `azure/login@v2` and the provided service principal.

- **ACR Login:**  
  Logs in to Azure Container Registry using `azure/docker-login@v2` with the service principal credentials.

- **Build Docker Image:**  
  Builds the Docker image using the specified Dockerfile and tags it for ACR.

- **Push Docker Image:**  
  Pushes the built image to the specified ACR repository.

### 2. Wait for Validation

- **Manual Approval:**  
  Uses `trstringer/manual-approval@v1` to require a manual approval before deployment.

### 3. Deploy Docker

- **Checkout code:**  
  Checks out the code again for deployment context.

- **Azure Login:**  
  Authenticates to Azure.

- **Deploy to Azure Container App:**  
  Uses Azure CLI to update the Container App with the new image.

---

## Example Usage

```yaml
jobs:
  call-docker-flow:
    uses: your-org/your-repo/.github/workflows/docker-flow.yml@main
    with:
      vmImageName: 'ubuntu-latest'
      resourceGroupName: 'my-resource-group'
      containerRegistry: 'myregistry.azurecr.io'
      containerAppName: 'my-app'
      dockerfilePath: './Dockerfile'
      imageRepository: 'my-image'
      tag: 'latest'
    secrets:
      AZURE_CREDENTIALS: ${{ secrets.AZURE_CREDENTIALS }}
```

## Requirements

- The service principal in AZURE_CREDENTIALS must have:
- AcrPush role on the ACR
- Permission to update the specified Container App
- The Dockerfile path and repository names must be correct and exist in your repo.
- The **ARC**, must be **Admin user**: Resource -> Settings -> Access Keys -> Admin User must be checked.