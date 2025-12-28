# Petclinic Infra Repository

This repository contains the infrastructure as code (IaC) for deploying the Spring Petclinic application and its PostgreSQL database using GitOps principles with ArgoCD and Argo Rollouts.

---

## Description

This repository contains Helm charts and ArgoCD application manifests for deploying the Spring Petclinic application and PostgreSQL database in Kubernetes.

- **Helm Charts**: Located in the `charts` directory, these charts define the Kubernetes resources needed for deploying the application and database.
- **Values Files**: Environment-specific configurations are located in the `environments` directory. These files contain custom values for dev and prod environments.
- **ArgoCD Applications**: Located in the `apps` directory, these manifests define the ArgoCD applications for deploying the Helm charts to the Kubernetes cluster.

---

## Prerequisites

- Kubernetes cluster
- ArgoCD installed in the cluster
- Argo Rollouts installed in the cluster
- kubeseal for encrypting secrets
- Helm for managing Kubernetes applications

---

## Setup Instructions

1. **Clone the Repository**:

   ```bash
   git clone https://github.com/romangulyako/petclinic-infra.git
   cd petclinic-infra
   ```
   
2. **Encrypt Secrets**:

  Use kubeseal to encrypt secrets for both dev and prod environments. Example for dev environment:
  
  ```bash
  kubectl create secret generic petclinic-db-dev --namespace=petclinic-dev \
  --from-literal=SPRING_DATASOURCE_USERNAME=dev_user \
  --from-literal=SPRING_DATASOURCE_PASSWORD=dev_password \
  --dry-run=client -o yaml > petclinic-db-dev-secret.yaml
  
  kubeseal --format yaml < petclinic-db-dev-secret.yaml > petclinic-db-dev-sealed-secret.yaml
  ```
  
  Copy the encrypted values to environments/dev/petclinic-values.yaml.
  
3. **Apply ArgoCD Applications**:

  Apply the ArgoCD application manifests to deploy the application and database:
  
  ```bash
  kubectl apply -n argocd -f apps/
  ```
  
## Usage

  - **Deploying a New Version**:
    - Push changes to the develop or main branch of the Spring Petclinic repository.
    - The CI/CD pipeline will build a new Docker image, update the Helm values file in this repository, and push the changes.
    - ArgoCD will detect the changes and deploy the new version of the application.
  
  - **Promoting a New Version**:
    Use Argo Rollouts to promote a new version of the application:
    
    ```bash
    kubectl argo rollouts promote petclinic -n petclinic-dev
    ```
    
  - **Rolling Back**:
    Use Argo Rollouts to roll back to a previous version of the application:
    
    ```bash
    kubectl argo rollouts undo petclinic -n petclinic-dev
    ```