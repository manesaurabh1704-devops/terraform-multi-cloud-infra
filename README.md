# 🏗️ Terraform Multi-Cloud Infrastructure

> Production-grade Terraform IaC for StudentSphere application.
> Provisions AWS EKS, Azure AKS, and GCP GKE — all three clouds.
> Part of the [multi-cloud-devops-studentsphere](https://github.com/manesaurabh1704-devops/multi-cloud-devops-studentsphere) project.

---

## 📁 Repository Structure

```
terraform-multi-cloud-infra/
├── aws/                        # AWS Infrastructure (Phase 4) ✅
│   ├── main.tf                 # AWS provider + Terraform config
│   ├── variables.tf            # All configurable variables
│   ├── vpc.tf                  # VPC, Subnets, IGW, Route Tables
│   ├── eks.tf                  # EKS Cluster, Node Group, IAM Roles
│   ├── ecr.tf                  # ECR Repositories + Lifecycle Policies
│   └── outputs.tf              # Cluster URL, VPC ID, ECR URLs
├── azure/                      # Azure Infrastructure (Phase 9) ✅
│   ├── main.tf                 # Azure provider config
│   ├── variables.tf            # Subscription ID, region, cluster config
│   ├── vnet.tf                 # VNet + Public/Private Subnet + NAT Gateway
│   ├── aks.tf                  # AKS Cluster — nodes in private subnet
│   └── outputs.tf              # Cluster name, endpoint, subnet IDs
├── gcp/                        # GCP Infrastructure (Phase 10) ✅
│   ├── main.tf                 # GCP provider config
│   ├── variables.tf            # Project ID, region, zone, cluster config
│   ├── vpc.tf                  # VPC + Public/Private Subnet + Cloud NAT
│   ├── gke.tf                  # GKE Cluster + Node Pool — private nodes
│   └── outputs.tf              # Cluster name, endpoint, project ID
├── screenshots/                # Proof of terraform plan/apply
└── README.md
```

---

## 🔄 Why Terraform / Why IaC?

```
Without Terraform (Manual):
  eksctl create cluster ...    (not reproducible — different every time)
  Manual VPC setup             (error-prone + no version control)
  No audit trail               (who changed what, when?)

With Terraform (IaC):
  terraform apply              (entire infra in 15 minutes — repeatable)
  Version controlled infra     (review changes like code — PR + approval)
  Same workflow for all clouds (AWS → Azure → GCP — learn once, use everywhere)
  terraform destroy            (clean teardown — no orphaned resources)
  terraform plan               (preview changes before applying — safe)
```

---

## ☁️ Cloud Coverage

| Cloud | Service | Region | Nodes | Resources | Status |
|---|---|---|---|---|---|
| AWS | EKS + VPC + ECR | ap-south-1 | t3.small x2 | 24 | ✅ Phase 4 Complete |
| Azure | AKS + VNet + NAT | West US 2 | Standard_B2s_v2 x2 | 9 | ✅ Phase 9 Complete |
| GCP | GKE + VPC + Cloud NAT | us-central1 | e2-medium x2 | 7 | ✅ Phase 10 Complete |

---

## 🏗️ Architecture — All 3 Clouds

### AWS EKS (Phase 4)
```
AWS Region: ap-south-1
    │
    ├── VPC (192.168.0.0/16)
    │   ├── Public Subnets  (x3) — EKS Nodes + LoadBalancer
    │   ├── Private Subnets (x3) — Future use
    │   ├── Internet Gateway
    │   └── Route Tables
    │
    ├── EKS Cluster (studentsphere-cluster)
    │   ├── Kubernetes v1.34
    │   ├── Node Group (t3.small x2)
    │   └── IAM Roles + Policies
    │
    └── ECR Repositories
        ├── studentsphere-backend
        └── studentsphere-frontend
```

### Azure AKS (Phase 9) — Production-Grade Private Nodes
```
Azure West US 2
    │
    ├── VNet (10.0.0.0/16)
    │   ├── Public Subnet  (10.0.1.0/24) ← Load Balancer only
    │   ├── Private Subnet (10.0.2.0/24) ← AKS Nodes (internet se hidden!)
    │   ├── Public IP (Static)
    │   └── NAT Gateway ← Nodes ka outbound internet
    │
    └── AKS Cluster (studentsphere-aks)
        ├── Kubernetes v1.35.1
        ├── Node Pool (Standard_B2s_v2 x2) ← private subnet mein
        └── SystemAssigned Identity
```

### GCP GKE (Phase 10) — Production-Grade Private Nodes
```
GCP us-central1
    │
    ├── VPC Network
    │   ├── Public Subnet  (10.0.1.0/24) ← Load Balancer only
    │   ├── Private Subnet (10.0.2.0/24) ← GKE Nodes (internet se hidden!)
    │   │   ├── Pods Range     (10.1.0.0/16)
    │   │   └── Services Range (10.2.0.0/16)
    │   ├── Cloud Router
    │   └── Cloud NAT ← Nodes ka outbound internet
    │
    └── GKE Cluster (studentsphere-gke)
        ├── Kubernetes v1.35.1
        ├── enable_private_nodes: true
        └── Node Pool (e2-medium x2)
```

---

## 📋 AWS Resources Created (24 total)

| Resource | Type | Description |
|---|---|---|
| aws_vpc | VPC | Main VPC with DNS enabled |
| aws_subnet (public) | Subnet x3 | Public subnets across 3 AZs |
| aws_subnet (private) | Subnet x3 | Private subnets across 3 AZs |
| aws_internet_gateway | IGW | Internet access for public subnets |
| aws_route_table | Route Table | Public traffic routing |
| aws_eks_cluster | EKS | Managed Kubernetes cluster |
| aws_eks_node_group | Node Group | t3.small worker nodes |
| aws_iam_role (cluster) | IAM Role | EKS cluster permissions |
| aws_iam_role (nodes) | IAM Role | EKS node permissions |
| aws_ecr_repository | ECR x2 | Backend + Frontend image repos |
| aws_ecr_lifecycle_policy | Policy x2 | Keep last 10 images |

---

## 📋 Azure Resources Created (9 total)

| Resource | Type | Description |
|---|---|---|
| azurerm_resource_group | Resource Group | Container for all resources |
| azurerm_virtual_network | VNet (10.0.0.0/16) | Main virtual network |
| azurerm_subnet (public) | Subnet (10.0.1.0/24) | Load Balancer only |
| azurerm_subnet (private) | Subnet (10.0.2.0/24) | AKS Nodes |
| azurerm_public_ip | Static Public IP | NAT Gateway outbound IP |
| azurerm_nat_gateway | NAT Gateway | Outbound internet for private nodes |
| azurerm_nat_gateway_public_ip_association | Association | Link NAT to Public IP |
| azurerm_subnet_nat_gateway_association | Association | Link NAT to Private Subnet |
| azurerm_kubernetes_cluster | AKS Cluster | Managed Kubernetes v1.35.1 |

---

## 📋 GCP Resources Created (7 total)

| Resource | Type | Description |
|---|---|---|
| google_compute_network | VPC Network | Main virtual network |
| google_compute_subnetwork (public) | Subnet (10.0.1.0/24) | Load Balancer only |
| google_compute_subnetwork (private) | Subnet (10.0.2.0/24) | GKE Nodes + Pod/Service ranges |
| google_compute_router | Cloud Router | For Cloud NAT |
| google_compute_router_nat | Cloud NAT | Outbound internet for private nodes |
| google_container_cluster | GKE Cluster | Managed Kubernetes v1.35.1 |
| google_container_node_pool | Node Pool | e2-medium worker nodes |

---

## ⚡ How to Use — AWS EKS

### Prerequisites

```bash
# Install Terraform
sudo apt install -y gnupg software-properties-common
wget -O- https://apt.releases.hashicorp.com/gpg | \
  sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
  https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install -y terraform

# Verify
terraform --version
```

Expected output:
```
Terraform v1.14.8
on linux_amd64
```

```bash
# Configure AWS CLI
aws configure
```

```
AWS Access Key ID:     YOUR_ACCESS_KEY
AWS Secret Access Key: YOUR_SECRET_KEY
Default region name:   ap-south-1
Default output format: json
```

### Step 1 — Clone + Navigate

```bash
git clone https://github.com/manesaurabh1704-devops/terraform-multi-cloud-infra.git
cd terraform-multi-cloud-infra/aws
```

### Step 2 — Terraform Init

```bash
terraform init
```

Expected output:
```
- Installing hashicorp/aws v5.100.0...
Terraform has been successfully initialized!
```

### Step 3 — Terraform Plan

```bash
terraform plan
```

Expected output:
```
Plan: 24 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + cluster_name      = "studentsphere-cluster"
  + cluster_version   = "1.34"
  + ecr_backend_url   = (known after apply)
  + ecr_frontend_url  = (known after apply)
  + vpc_id            = (known after apply)
```

### Step 4 — Terraform Apply

```bash
terraform apply -auto-approve
```

Expected output:
```
aws_vpc.main: Creating...
aws_eks_cluster.main: Creation complete after 12m
aws_eks_node_group.main: Creation complete after 2m

Apply complete! Resources: 24 added, 0 changed, 0 destroyed.

Outputs:
cluster_name     = "studentsphere-cluster"
ecr_backend_url  = "207457247776.dkr.ecr.ap-south-1.amazonaws.com/studentsphere-backend"
ecr_frontend_url = "207457247776.dkr.ecr.ap-south-1.amazonaws.com/studentsphere-frontend"
vpc_id           = "vpc-XXXXXXXXXXXXXXXXX"
```

### Step 5 — Configure kubectl

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name studentsphere-cluster

kubectl get nodes
```

Expected output:
```
NAME                                            STATUS   ROLES    AGE
ip-192-168-62-10.ap-south-1.compute.internal    Ready    <none>   5m
ip-192-168-86-169.ap-south-1.compute.internal   Ready    <none>   5m
```

### Step 6 — Destroy (To Save Cost)

```bash
terraform destroy -auto-approve
```

Expected output:
```
Destroy complete! Resources: 24 destroyed.
```

---

## ⚡ How to Use — Azure AKS

### Prerequisites

```bash
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Verify
az --version
```

Expected output:
```
azure-cli 2.85.0
```

```bash
# Login to Azure
az login --use-device-code
```

```
Open browser: https://microsoft.com/devicelogin
Enter the code shown in terminal
```

```bash
# Verify subscription
az account show --output table

# Register required providers
az provider register --namespace Microsoft.ContainerService
az provider register --namespace Microsoft.Network
az provider register --namespace Microsoft.Compute

# Verify
az provider show -n Microsoft.ContainerService --query "registrationState"
```

Expected output:
```
"Registered"
```

### Step 1 — Navigate to Azure folder

```bash
cd terraform-multi-cloud-infra/azure
```

### Step 2 — Terraform Init

```bash
terraform init
```

Expected output:
```
- Installing hashicorp/azurerm v3.117.1...
Terraform has been successfully initialized!
```

### Step 3 — Terraform Plan

```bash
terraform plan 2>&1 | tail -15
```

Expected output:
```
Plan: 9 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + cluster_endpoint  = (sensitive value)
  + cluster_name      = "studentsphere-aks"
  + private_subnet_id = (known after apply)
  + public_subnet_id  = (known after apply)
  + resource_group    = "studentsphere-rg"
```

### Step 4 — Terraform Apply

```bash
terraform apply -auto-approve
```

Expected output:
```
azurerm_resource_group.main: Creation complete after 16s
azurerm_virtual_network.main: Creation complete after 12s
azurerm_subnet.private: Creation complete after 7s
azurerm_nat_gateway.main: Creation complete after 13s
azurerm_subnet_nat_gateway_association.private: Creation complete after 7s
azurerm_kubernetes_cluster.main: Creation complete after 4m17s

Apply complete! Resources: 9 added, 0 changed, 0 destroyed.

Outputs:
cluster_name   = "studentsphere-aks"
resource_group = "studentsphere-rg"
```

### Step 5 — Configure kubectl

```bash
az aks get-credentials \
  --resource-group studentsphere-rg \
  --name studentsphere-aks \
  --overwrite-existing

# Verify
kubectl get nodes
```

Expected output:
```
NAME                              STATUS   ROLES    AGE   VERSION
aks-default-xxxx-vmss000000       Ready    <none>   5m    v1.35.1
aks-default-xxxx-vmss000001       Ready    <none>   5m    v1.35.1
```

### Step 6 — Give AKS Permission for LoadBalancer

```bash
# Required for LoadBalancer External IP to work
AKS_IDENTITY=$(az aks show \
  --resource-group studentsphere-rg \
  --name studentsphere-aks \
  --query "identity.principalId" -o tsv)

VNET_ID=$(az network vnet show \
  --resource-group studentsphere-rg \
  --name studentsphere-vnet \
  --query id -o tsv)

az role assignment create \
  --assignee $AKS_IDENTITY \
  --role "Network Contributor" \
  --scope $VNET_ID

echo "Permission assigned!"
```

### Step 7 — Destroy (To Save Cost)

```bash
terraform destroy -auto-approve
```

Expected output:
```
Destroy complete! Resources: 9 destroyed.
```

---

## ⚡ How to Use — GCP GKE

### Prerequisites

```bash
# Install gcloud CLI
curl https://sdk.cloud.google.com | bash
source ~/.bashrc

# Verify
gcloud --version
```

Expected output:
```
Google Cloud SDK 564.0.0
```

```bash
# Login to GCP
gcloud auth login --no-launch-browser
gcloud auth application-default login --no-launch-browser
```

```bash
# Set project
gcloud projects list
gcloud config set project YOUR_PROJECT_ID

# Enable required APIs
gcloud services enable container.googleapis.com
gcloud services enable compute.googleapis.com

# Install GKE auth plugin
gcloud components install gke-gcloud-auth-plugin

# Verify
gke-gcloud-auth-plugin --version
```

### Step 1 — Navigate to GCP folder

```bash
cd terraform-multi-cloud-infra/gcp
```

### Step 2 — Terraform Init

```bash
terraform init
```

Expected output:
```
- Installing hashicorp/google v5.x.x...
Terraform has been successfully initialized!
```

### Step 3 — Terraform Plan

```bash
terraform plan 2>&1 | tail -15
```

Expected output:
```
Plan: 7 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + cluster_endpoint = (sensitive value)
  + cluster_name     = "studentsphere-gke"
  + project_id       = "project-a3e71bc3-7f01-47c1-ae7"
  + region           = "us-central1"
```

### Step 4 — Terraform Apply

```bash
terraform apply -auto-approve
```

Expected output:
```
google_compute_network.main: Creating...
google_compute_subnetwork.private: Creation complete
google_compute_router_nat.main: Creation complete
google_container_cluster.main: Creation complete after 10m
google_container_node_pool.main: Creation complete after 2m

Apply complete! Resources: 7 added, 0 changed, 0 destroyed.

Outputs:
cluster_name = "studentsphere-gke"
project_id   = "project-a3e71bc3-7f01-47c1-ae7"
region       = "us-central1"
```

### Step 5 — Configure kubectl

```bash
gcloud container clusters get-credentials studentsphere-gke \
  --zone us-central1-a \
  --project YOUR_PROJECT_ID

# Verify
kubectl get nodes
```

Expected output:
```
NAME                                                  STATUS   ROLES    AGE   VERSION
gke-studentsphere-gk-studentsphere-no-xxxx-xxxx       Ready    <none>   5m    v1.35.1-gke
gke-studentsphere-gk-studentsphere-no-xxxx-xxxx       Ready    <none>   5m    v1.35.1-gke
```

### Step 6 — Destroy (To Save Cost)

```bash
terraform destroy -auto-approve
```

Expected output:
```
Destroy complete! Resources: 7 destroyed.
```

---

## 🔧 Customization

### AWS — variables.tf
```hcl
variable "aws_region"         { default = "us-east-1"  }  # Change region
variable "node_instance_type" { default = "t3.medium"   }  # Upgrade VM
variable "node_desired_size"  { default = 3             }  # More nodes
```

### Azure — variables.tf
```hcl
variable "location"           { default = "East US"          }  # Change region
variable "node_size"          { default = "Standard_B2s_v2"  }  # VM size
variable "node_count"         { default = 2                  }  # Node count
variable "kubernetes_version" { default = "1.35.1"           }  # K8s version
```

### GCP — variables.tf
```hcl
variable "region"            { default = "us-central1"   }  # Change region
variable "zone"              { default = "us-central1-a"  }  # Change zone
variable "node_machine_type" { default = "e2-medium"      }  # VM size
variable "node_count"        { default = 2                }  # Node count
```

---

## 📸 Output / Proof

### Terraform Init
![Terraform Init](screenshots/01-terraform-init.png)

### Terraform Plan — 24 Resources
![Terraform Plan](screenshots/02-terraform-plan.png)

### GitHub — All Terraform Files
![GitHub Files](screenshots/03-github-terraform-files.png)

---

## 🐛 Troubleshooting

### AWS
```
Error: No valid credential sources found
Fix: aws configure

Error: InvalidParameterCombination - instance type not eligible
Fix: Use t3.small for new/free accounts

Error: resource already exists
Fix: terraform import aws_eks_cluster.main studentsphere-cluster

Error: provider version conflict
Fix: terraform init -upgrade
```

### Azure
```
Error: VM size not allowed in subscription
Fix: Use Standard_B2s_v2
     az vm list-skus --location westus2 --size Standard_B

Error: K8sVersionNotSupported
Fix: az aks get-versions --location westus2 --output table
     Use latest available (e.g. 1.35.1)

Error: ServiceCidrOverlapExistingSubnetsCidr
Fix: In aks.tf network_profile:
     service_cidr   = "172.16.0.0/16"
     dns_service_ip = "172.16.0.10"

Error: LoadBalancer External IP stuck at pending
Fix: Assign Network Contributor role (See Step 6)
```

### GCP
```
Error: gke-gcloud-auth-plugin not found
Fix: gcloud components install gke-gcloud-auth-plugin

Error: Billing account not found
Fix: Enable billing at console.cloud.google.com/billing

Error: Insufficient CPU for pods
Fix: kubectl scale deployment backend --replicas=1 -n studentsphere
```

---

## 🔗 Related Repositories

| Repository | Purpose |
|---|---|
| [multi-cloud-devops-studentsphere](https://github.com/manesaurabh1704-devops/multi-cloud-devops-studentsphere) | Main project — Full DevOps system |
| [kubernetes-production-setup](https://github.com/manesaurabh1704-devops/kubernetes-production-setup) | Kubernetes manifests |
| [ci-cd-devops-pipelines](https://github.com/manesaurabh1704-devops/ci-cd-devops-pipelines) | Jenkins CI/CD pipelines |
| [monitoring-observability-stack](https://github.com/manesaurabh1704-devops/monitoring-observability-stack) | Prometheus + Grafana |
| [devops-security-secrets](https://github.com/manesaurabh1704-devops/devops-security-secrets) | RBAC + Security |

---

## 👨‍💻 Author
**Saurabh Mane** — DevOps Engineer
- GitHub: [@manesaurabh1704-devops](https://github.com/manesaurabh1704-devops)

---

> ⭐ Star this repo if you find it helpful!
