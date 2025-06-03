# GitHub Actions Workflows: docker-java-build-publish & docker-java-deploy

These workflow templates automate the process of building, publishing, and deploying Java applications as Docker images to Azure Container Apps using Azure Container Registry (ACR).  
They are designed to be **reusable** via `workflow_call` and require specific inputs and secrets.

---

## Workflow 1: `docker-java-build-publish`

### **Purpose**
Builds your Java project, creates a Docker image, and pushes it to Azure Container Registry.

### **Stages**
1. **Checkout Code**
2. **Set Up Java**
3. **Build Java Project**
4. **Azure Login**
5. **ACR Login**
6. **Build Docker Image**
7. **Push Docker Image**

### **Inputs**

| Name                | Type   | Required | Description                                   |
|---------------------|--------|----------|-----------------------------------------------|
| `vmImageName`       | string | Yes      | VM image for the runner (e.g., `ubuntu-latest`) |
| `containerRegistry` | string | Yes      | Azure Container Registry login server (e.g., `myregistry.azurecr.io`) |
| `dockerfilePath`    | string | Yes      | Path to the Dockerfile (e.g., `./Dockerfile`) |
| `imageRepository`   | string | Yes      | Name of the image repository in ACR           |
| `tag`               | string | Yes      | Docker image tag (e.g., `latest`)             |

### **Secrets**

| Name                | Required | Description                                      |
|---------------------|----------|--------------------------------------------------|
| `AZURE_CREDENTIALS` | Yes      | Azure service principal credentials as JSON. Must have permissions to push to ACR. |

### **Steps Breakdown**

- **Checkout code:**  
  Uses `actions/checkout@v4` to get the repository content.

- **Set up Java:**  
  Uses `actions/setup-java@v4` to install the required JDK (e.g., Temurin 23).

- **Build Java project:**  
  Runs `./gradlew build -x test` (or similar) to generate the JAR file.

- **Azure Login:**  
  Authenticates to Azure using `azure/login@v2` and the provided service principal.

- **ACR Login:**  
  Logs in to Azure Container Registry using `azure/docker-login@v2`.

- **Build Docker Image:**  
  Builds the Docker image using the specified Dockerfile and tags it for ACR.

- **Push Docker Image:**  
  Pushes the built image to the specified ACR repository.

---

## Workflow 2: `docker-java-deploy`

### **Purpose**
Deploys a Docker image from Azure Container Registry to an Azure Container App.

### **Stages**
1. **Checkout Code**
2. **Azure Login**
3. **Deploy to Azure Container App**

### **Inputs**

| Name                | Type   | Required | Description                                   |
|---------------------|--------|----------|-----------------------------------------------|
| `vmImageName`       | string | Yes      | VM image for the runner (e.g., `ubuntu-latest`) |
| `resourceGroupName` | string | Yes      | Azure Resource Group name                     |
| `containerRegistry` | string | Yes      | Azure Container Registry login server (e.g., `myregistry.azurecr.io`) |
| `containerAppName`  | string | Yes      | Azure Container App name                      |
| `imageRepository`   | string | Yes      | Name of the image repository in ACR           |
| `tag`               | string | Yes      | Docker image tag (e.g., `latest`)             |

### **Secrets**

| Name                | Required | Description                                      |
|---------------------|----------|--------------------------------------------------|
| `AZURE_CREDENTIALS` | Yes      | Azure service principal credentials as JSON. Must have permissions to update Container Apps. |

### **Steps Breakdown**

- **Checkout code:**  
  Uses `actions/checkout@v4` to get the repository content.

- **Azure Login:**  
  Authenticates to Azure using `azure/login@v2`.

- **Deploy to Azure Container App:**  
  Uses Azure CLI to update the Container App with the new image from ACR.

---

## Example Usage

### Build & Publish

```yaml
jobs:
  build_and_publish:
    uses: your-org/your-repo/.github/workflows/docker-java-build-publish.yml@main
    with:
      vmImageName: 'ubuntu-latest'
      containerRegistry: 'myregistry.azurecr.io'
      dockerfilePath: './Dockerfile'
      imageRepository: 'my-image'
      tag: 'latest'
    secrets:
      AZURE_CREDENTIALS: ${{ secrets.AZURE_CREDENTIALS }}
```

### Deploy

```yaml
jobs:
  deploy:
    uses: your-org/your-repo/.github/workflows/docker-java-deploy.yml@main
    with:
      vmImageName: 'ubuntu-latest'
      resourceGroupName: 'my-resource-group'
      containerRegistry: 'myregistry.azurecr.io'
      containerAppName: 'my-app'
      imageRepository: 'my-image'
      tag: 'latest'
    secrets:
      AZURE_CREDENTIALS: ${{ secrets.AZURE_CREDENTIALS }}
```

---

## Requirements

- The service principal in `AZURE_CREDENTIALS` must have:
  - **AcrPush** role on the ACR (for build/publish)
  - **AcrPull** role for the Container App's managed identity (for deploy)
  - Permission to update the specified Container App
- The Dockerfile path and repository names must be correct and exist in your repo.
- The Container App must be configured to use a managed identity for ACR authentication.

---

## Troubleshooting

- **Authentication errors:**  
  Double-check the Azure Service Principal credentials and permissions.
- **Image not found:**  
  Ensure the image was built and pushed to ACR before deploying.
- **Container App cannot pull image:**  
  Ensure the Container App's managed identity has the AcrPull role on the ACR and is enabled.

---

**Tip:**  
Never commit secrets to your repository. Always use GitHub Secrets for sensitive information.