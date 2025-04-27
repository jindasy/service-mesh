# Consul Service Mesh with Microservices Application

## Project Overview

This project demonstrates setting up an **e-commerce microservices application** across **multiple Kubernetes clusters** in **multi-cloud environments** using **Consul Service Mesh** for **failover**.

It integrates:

- Infrastructure and deployment strategies from **[TechWorld with Nana's Consul Crash Course](https://gitlab.com/twn-youtube/consul-crash-course)**.
- The application source code from **[Google Cloud Platform Microservices Demo](https://github.com/GoogleCloudPlatform/microservices-demo)**.



## Codes and Scripts (from TechWorld with Nana - Consul Crash Course)

### Terraform Scripts (for AWS EKS Cluster)

Located under `terraform/`:

- **main.tf**:  
  Defines the AWS infrastructure needed for an EKS cluster.

- **variables.tf**:  
  Defines variables (like region, cluster name) used across Terraform files.

- **terraform.tfvars**:  
  File containing actual variable values.

**Purpose**:  
Provision a ready-to-use Kubernetes cluster (AWS EKS) to deploy Consul and applications.

---

### Kubernetes Manifests

Located under `kubernetes/`:

- **config.yaml**:  
  Standard Kubernetes deployments and services for the microservices application.  
  Initially deployed before integrating Consul.

- **config-consul.yaml**:  
  Kubernetes deployment modified to inject Consul into each microservice pod using Consul annotations.

- **consul-mesh-gateway.yaml**:  
  Defines Consul Mesh Gateways, allowing communication between services running on different clusters/clouds.

- **exported-service.yaml**:  
  Used in the secondary cluster (LKE) to **export services** to Consul so that other clusters can discover them.

- **service-resolver.yaml**:  
  Customizes service discovery in Consul, telling Consul how to resolve traffic for specific services.



## How to Setup and Deployment

### 1. Provision AWS EKS Cluster (First Cluster)

```bash
# Initialize Terraform for AWS EKS
cd terraform
terraform init

# Apply Terraform with specified variables
terraform apply -var-file terraform.tfvars
```

### 2. Connect to AWS EKS Cluster
```bash
# Configure AWS credentials
aws configure

# Update kubeconfig to connect to EKS
aws eks update-kubeconfig --region <your-region> --name myapp-eks-cluster

# Check EKS nodes
kubectl get node
```
### 3. Deploy Application to EKS
```bash
# Apply configuration with Consul sidecar proxies
kubectl apply -f config-consul.yaml

# Verify deployments
kubectl get pod
kubectl get svc
```

### 4. Install Consul on EKS Using Helm
```bash
# Add HashiCorp Helm repository
helm repo add hashicorp https://helm.releases.hashicorp.com

# Install Consul with specific datacenter name
helm install eks hashicorp/consul --version 1.0.0 --values consul-values.yaml --set global.datacenter=eks

# Verify Consul installation
kubectl get pod
kubectl get all
```


### 5. Setup Second Cluster (LKE) and Install Consul
Create second cluster on another cloud provider.
```bash
# Install Consul with datacenter name "lke"
helm install lke hashicorp/consul --version 1.0.0 --values consul-values.yaml --set global.datacenter=lke

# Check all Kubernetes objects
kubectl get all

# Check installed CRDs (Custom Resource Definitions)
kubectl get crd
kubectl get crd | grep consul

# Verify pods
kubectl get pod
```

### 7. Deploy Second Application
```bash
# Apply the updated configuration with Consul integration
kubectl apply -f config-consul.yaml

# Verify deployments
kubectl get pod
kubectl get svc
```

### 8. Enable Consul Mesh Gateway for Cross-Cluster Communication
On both EKS and LKE clusters:
```bash
# Apply Mesh Gateway configuration
kubectl apply -f consul-mesh-gateway.yaml

# Verify mesh gateway deployment
kubectl get mesh
```

### 9. Setup Service Failover
On EKS:
```bash
# Delete specific service (e.g., shippingservice) to simulate failure
kubectl delete deployment shippingservice

# Verify pods
kubectl get pod

# Reapply config if needed
kubectl apply -f config-consul.yaml
```
On LKE:

```bash
# Export service to be discoverable across clusters
kubectl apply -f exported-service.yaml
```
On EKS:
```bash
# Apply service resolver configuration
kubectl apply -f service-resolver.yaml

# Re-deploy or clean up as needed
kubectl delete deployment shippingservice
```

## Acknowledgements
- Google Cloud Platform — for the [Microservices Demo](https://github.com/GoogleCloudPlatform/microservices-demo)

- TechWorld with Nana — for the [Consul Service Mesh Tutorial for Beginners [Crash Course]](https://www.youtube.com/watch?v=s3I1kKKfjtQ)

