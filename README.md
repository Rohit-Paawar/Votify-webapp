# 🗳️ VOTIFY: Automated CI/CD for the Voting App

This project implements a robust, secure, and fully automated Continuous Integration and Continuous Delivery (CI/CD) pipeline for the Docker Example Voting App, utilizing **Azure DevOps** for orchestration and **Azure Kubernetes Service (AKS)** for deployment.

## 1\. Project Architecture and Toolchain

The VOTIFY application is based on the five microservices from the Docker Example Voting App, orchestrated by a modern cloud-native toolchain:

| Component | Technology | Role | Azure Resource |
| :--- | :--- | :--- | :--- |
| **Source Control** | Git | Hosts the source code and K8s manifests. | Azure Repos (VOTIFY) |
| **CI/CD Platform** | YAML Pipelines | Automates build, test, and push processes. | Azure DevOps Pipelines |
| **Container Registry** | Docker | Stores the final built images. | **Azure Container Registry** (`votifyacr`) |
| **Deployment Target** | Kubernetes | Orchestrates containers in production. | **Azure Kubernetes Service** (`votifyaks`) |
| **CI Runner** | Docker / Linux | Executes build jobs. | Self-Hosted Agent Pool (`Votingapp`) |

## 2\. Continuous Integration (CI) Implementation

The CI process ensures that every code change is independently built, tested, and containerized.

![Repo overview](https://github.com/user-attachments/assets/943c6dd1-aea9-4a22-a09a-5b28bd7439dd)

### 2.1 Dedicated Pipelines and Parallelism

To maintain the microservices principle, a separate pipeline is configured for each custom application component:

  * **Result Pipeline**
  * **Worker Pipeline**
  * **Vote Pipeline**

![Pipelines](https://github.com/user-attachments/assets/898f9bd5-95eb-4a36-9141-978e4e3f3217)


This parallel structure, as seen in the pipeline runs, allows for faster integration and independent releases for each microservice.

### 2.2 Self-Hosted Agent Pool

All CI jobs run on a dedicated **Self-Hosted Agent Pool** named **`Votingapp`**. This pool is vital for executing Docker build commands and potentially running specialized tests.

The agent is running on an Azure Virtual Machine Scale Set instance, providing a controlled and scalable environment:

| Agent Pool Name | Agent Host VMSS | Project |
| :--- | :--- | :--- |
| **`Votingapp`** | `aks-agentpool-28478938-vmss` | VOTIFY |

![Agentpool](https://github.com/user-attachments/assets/d257b2f0-1df3-400c-9bc9-7d5800f9853a)


### 2.3 Secure Registry Integration

The built Docker images are pushed to the **Azure Container Registry** (`votifyacr`).

  * **Registry Target:** `votifyacr` (Type: Container registry)
  * **Azure Resource:** `votifyaks` (Type: Kubernetes service)
<img width="1918" height="1010" alt="portal" src="https://github.com/user-attachments/assets/40fca028-672f-40ac-85f8-51023ad1f57b" />


Access to this registry from Azure DevOps is secured using an **Azure Resource Manager Service Connection** named **`votify-acr`**. This connection utilizes **Workload Identity Federation (OIDC)**, a modern security standard that avoids storing long-lived credentials, making the pipeline highly secure.

## 3\. Continuous Deployment (CD) and Verification

The CD process takes the validated images from ACR and deploys them to the **`votifyaks`** cluster.

### 3.1 Deployment Artifacts

The deployment is managed by standard Kubernetes manifest files, located in the **`k8s-specifications`** directory within the repository.

  * **Key Files:** These YAML files define the Deployment, Service, and ConfigMap objects necessary to run the 5-tier application.

![Service connection](https://github.com/user-attachments/assets/85021740-1ebc-44a7-a4e2-48270f77f509)


### 3.2 Live Deployment Verification

The final step confirms the successful deployment of the updated application to the AKS environment via the Azure Cloud Shell.

The following commands confirm the cluster's health and the status of all application Pods in the target namespace (`votify-m`):

```bash
# Verify the Kubernetes cluster node status
azureuser@votingapp-agent:~/GitOps/VOTIFY$ kubectl get nodes -o wide
NAME                        STATUS   ROLES   AGE    VERSION    INTERNAL-IP     EXTERNAL-IP   OS-IMAGE
aks-agentpool-28478938-..   Ready    <none>  6h58m  v1.32.7    10.224.8.4      20.197.36.53  Ubuntu 22.04.5 LTS

# Verify the status of the application Pods in the 'votify-m' namespace
azureuser@votingapp-agent:~/GitOps/VOTIFY$ kubectl get po -n votify-m
NAME                   READY   STATUS    RESTARTS   AGE
db-7457f466dd-d9ghk    1/1     Running   0          17h
redis-6c5fb9cwb7-kdd4x 1/1     Running   0          17h
result-c7c498dc7-57wgd 1/1     Running   0          28m  # Latest deployment
vote-d4bdc94fbc7-pfx2q 1/1     Running   0          97m
worker-5bbf648984-lw95q 1/1     Running   0          81m
```
<img width="1918" height="546" alt="pod node" src="https://github.com/user-attachments/assets/1b01d388-c1d8-4ec5-914b-756625f0cc40" />


**Conclusion:** All five microservice pods (`db`, `redis`, `result`, `vote`, `worker`) are confirmed to be in the **Running** state, indicating a successful and fully distributed application deployment via the automated CI/CD pipeline.

-----
