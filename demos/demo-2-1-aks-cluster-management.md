# Demo 2.1: AKS Cluster Creation and Management (15 minutes)

## Overview
This comprehensive demo walks through creating, configuring, and managing an Azure Kubernetes Service (AKS) cluster. Participants will learn about cluster configuration options, node pool management, and essential cluster operations.

## Prerequisites
- Azure CLI installed and authenticated
- kubectl installed
- Azure subscription with appropriate permissions
- Basic understanding of Azure resource management

## Learning Objectives
- Create and configure AKS clusters using Azure CLI
- Understand cluster configuration options and their implications
- Manage node pools and scaling
- Connect to and interact with AKS clusters
- Implement monitoring and logging

---

## Step-by-Step Instructions

### Step 1: Azure Environment Setup (2 minutes)

1. **Verify Azure CLI installation and login:**
   ```bash
   az --version
   az account show
   ```

2. **Login to Azure (if not already authenticated):**
   ```bash
   az login
   ```

3. **Set the desired subscription:**
   ```bash
   az account list --output table
   az account set --subscription "<subscription-name-or-id>"
   ```

4. **Register required resource providers:**
   ```bash
   az provider register --namespace Microsoft.ContainerService
   az provider register --namespace Microsoft.OperationalInsights
   ```

### Step 2: Create Resource Group (1 minute)

1. **Create a resource group for the AKS resources:**
   ```bash
   az group create \
     --name aks-demo-rg \
     --location eastus
   ```

2. **Verify resource group creation:**
   ```bash
   az group show --name aks-demo-rg --output table
   ```

### Step 3: AKS Cluster Creation with Basic Configuration (3 minutes)

1. **Create a basic AKS cluster:**
   ```bash
   az aks create \
     --resource-group aks-demo-rg \
     --name aks-demo-cluster \
     --node-count 2 \
     --node-vm-size Standard_DS2_v2 \
     --enable-addons monitoring \
     --generate-ssh-keys \
     --enable-managed-identity
   ```

2. **Explain the parameters:**
   - `--node-count`: Initial number of nodes in default node pool
   - `--node-vm-size`: VM size for worker nodes
   - `--enable-addons monitoring`: Enables Azure Monitor for containers
   - `--generate-ssh-keys`: Creates SSH key pair for node access
   - `--enable-managed-identity`: Uses managed identity for cluster authentication

3. **Monitor cluster creation progress:**
   ```bash
   # In a separate terminal, you can watch the deployment
   az aks list --resource-group aks-demo-rg --output table
   ```

### Step 4: Cluster Connection and Verification (2 minutes)

1. **Get cluster credentials:**
   ```bash
   az aks get-credentials \
     --resource-group aks-demo-rg \
     --name aks-demo-cluster
   ```

2. **Verify cluster connectivity:**
   ```bash
   kubectl cluster-info
   kubectl get nodes
   kubectl get nodes -o wide
   ```

3. **Check all system pods:**
   ```bash
   kubectl get pods --all-namespaces
   kubectl get pods -n kube-system
   ```

4. **View cluster configuration:**
   ```bash
   kubectl config current-context
   az aks show --resource-group aks-demo-rg --name aks-demo-cluster --output table
   ```

### Step 5: Advanced Cluster Configuration (3 minutes)

1. **Create an advanced AKS cluster with additional features:**
   ```bash
   az aks create \
     --resource-group aks-demo-rg \
     --name aks-advanced-cluster \
     --node-count 1 \
     --min-count 1 \
     --max-count 5 \
     --enable-cluster-autoscaler \
     --network-plugin azure \
     --service-cidr 10.0.0.0/16 \
     --dns-service-ip 10.0.0.10 \
     --docker-bridge-address 172.17.0.1/16 \
     --enable-addons monitoring,azure-policy \
     --enable-managed-identity \
     --zones 1 2 3 \
     --kubernetes-version 1.28.3 \
     --generate-ssh-keys
   ```

2. **Explain advanced parameters:**
   - `--enable-cluster-autoscaler`: Automatic node scaling
   - `--network-plugin azure`: Azure CNI networking
   - `--service-cidr`: Internal service IP range
   - `--zones`: Availability zone distribution
   - `--kubernetes-version`: Specific Kubernetes version

### Step 6: Node Pool Management (2 minutes)

1. **List existing node pools:**
   ```bash
   az aks nodepool list \
     --resource-group aks-demo-rg \
     --cluster-name aks-demo-cluster \
     --output table
   ```

2. **Add a new node pool for specific workloads:**
   ```bash
   az aks nodepool add \
     --resource-group aks-demo-rg \
     --cluster-name aks-demo-cluster \
     --name userpool \
     --node-count 1 \
     --node-vm-size Standard_DS3_v2 \
     --node-taints workload=user:NoSchedule \
     --labels workload=user
   ```

3. **Scale a node pool:**
   ```bash
   az aks nodepool scale \
     --resource-group aks-demo-rg \
     --cluster-name aks-demo-cluster \
     --name nodepool1 \
     --node-count 3
   ```

4. **View node pool details:**
   ```bash
   kubectl get nodes --show-labels
   kubectl describe nodes
   ```

### Step 7: Cluster Monitoring and Operations (2 minutes)

1. **Check cluster health and status:**
   ```bash
   az aks show \
     --resource-group aks-demo-rg \
     --name aks-demo-cluster \
     --query "powerState" \
     --output table
   ```

2. **Get available Kubernetes versions:**
   ```bash
   az aks get-versions \
     --location eastus \
     --output table
   ```

3. **Upgrade cluster (demonstration - don't execute):**
   ```bash
   # Check upgrade availability
   az aks get-upgrades \
     --resource-group aks-demo-rg \
     --name aks-demo-cluster \
     --output table
   
   # Upgrade cluster (commented out for demo)
   # az aks upgrade \
   #   --resource-group aks-demo-rg \
   #   --name aks-demo-cluster \
   #   --kubernetes-version 1.28.4
   ```

4. **Enable/disable cluster features:**
   ```bash
   # Enable Azure Policy add-on
   az aks enable-addons \
     --resource-group aks-demo-rg \
     --name aks-demo-cluster \
     --addons azure-policy
   ```

### Step 8: Cleanup and Resource Management (1 minute)

1. **Stop cluster to save costs (optional):**
   ```bash
   az aks stop \
     --resource-group aks-demo-rg \
     --name aks-demo-cluster
   ```

2. **Start cluster when needed:**
   ```bash
   az aks start \
     --resource-group aks-demo-rg \
     --name aks-demo-cluster
   ```

3. **Delete specific node pool:**
   ```bash
   az aks nodepool delete \
     --resource-group aks-demo-rg \
     --cluster-name aks-demo-cluster \
     --name userpool \
     --no-wait
   ```

4. **Delete entire cluster (demonstration only):**
   ```bash
   # Uncomment to actually delete
   # az aks delete \
   #   --resource-group aks-demo-rg \
   #   --name aks-demo-cluster \
   #   --yes --no-wait
   ```

---

## AKS Cluster Configuration Options

### Networking
```bash
# Basic networking (kubenet - deprecated)
--network-plugin kubenet

# Advanced networking (Azure CNI - recommended)
--network-plugin azure
--service-cidr 10.0.0.0/16
--dns-service-ip 10.0.0.10
--docker-bridge-address 172.17.0.1/16
```

### Security and Identity
```bash
# Managed Identity (recommended)
--enable-managed-identity

# Service Principal (legacy)
--service-principal <app-id>
--client-secret <secret>

# Azure Active Directory integration
--enable-aad
--aad-admin-group-object-ids <group-id>
```

### Scaling and Availability
```bash
# Auto-scaling
--enable-cluster-autoscaler
--min-count 1
--max-count 10

# Availability zones
--zones 1 2 3

# Node pools
--nodepool-name <name>
--node-vm-size <size>
--node-count <count>
```

### Add-ons and Features
```bash
# Monitoring
--enable-addons monitoring

# Azure Policy
--enable-addons azure-policy

# Application Gateway Ingress Controller
--enable-addons ingress-appgw
```

---

## Monitoring and Troubleshooting Commands

### Cluster Status
```bash
# Cluster information
az aks show --resource-group <rg> --name <cluster-name>
kubectl cluster-info
kubectl get componentstatuses

# Node status
kubectl get nodes -o wide
kubectl describe node <node-name>
kubectl top nodes
```

### Networking
```bash
# Network configuration
kubectl get services --all-namespaces
kubectl get endpoints
kubectl get networkpolicies
```

### Logs and Events
```bash
# Cluster events
kubectl get events --sort-by='.lastTimestamp'

# System pod logs
kubectl logs -n kube-system <pod-name>

# Azure activity logs
az monitor activity-log list --resource-group <rg>
```

---

## Best Practices Checklist

### Security
- ✅ Use managed identity instead of service principal
- ✅ Enable Azure Active Directory integration
- ✅ Implement network policies
- ✅ Use private clusters for sensitive workloads
- ✅ Enable audit logging

### Networking
- ✅ Use Azure CNI for advanced networking features
- ✅ Plan IP address ranges carefully
- ✅ Implement network security groups
- ✅ Use Application Gateway for ingress

### Scalability
- ✅ Enable cluster autoscaler
- ✅ Use multiple node pools for different workloads
- ✅ Distribute across availability zones
- ✅ Set appropriate resource quotas

### Operations
- ✅ Enable monitoring and logging
- ✅ Implement backup strategies
- ✅ Plan for cluster upgrades
- ✅ Use infrastructure as code (ARM/Bicep templates)

---

## Key Takeaways

1. **AKS simplifies Kubernetes management** by handling the control plane
2. **Network planning is crucial** for production deployments
3. **Node pools enable workload isolation** and specialized configurations
4. **Monitoring and logging are essential** for operational visibility
5. **Security should be designed from the start** with proper identity and network controls

## Discussion Points

- **Cluster sizing**: How to determine appropriate node counts and VM sizes
- **Networking models**: When to use Azure CNI vs kubenet
- **Cost optimization**: Strategies for managing AKS costs
- **Upgrade strategies**: Planning and executing cluster upgrades

---

## Next Steps
- In Demo 3.1, we'll deploy applications to this AKS cluster
- Explore advanced AKS features like virtual nodes and Windows containers
- Learn about AKS integration with other Azure services

## Troubleshooting

**Common Issues:**
- **Authentication failures**: Check Azure CLI login and permissions
- **Network connectivity**: Verify subnet configurations and NSG rules
- **Resource quotas**: Ensure subscription has adequate quota for VM sizes
- **Version compatibility**: Check supported Kubernetes versions

**Useful commands for troubleshooting:**
```bash
az aks check-acr --resource-group <rg> --name <cluster> --acr <acr-name>
az aks kanalyze --resource-group <rg> --name <cluster>
kubectl get events --sort-by='.lastTimestamp' --all-namespaces
```