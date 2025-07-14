# Azure Kubernetes Service (AKS) Workshop - 3-Hour Session Agenda

## Workshop Overview
**Duration:** 3 Hours  
**Target Audience:** Developers, DevOps Engineers, Cloud Architects  
**Prerequisites:** Basic Azure knowledge, familiarity with command line tools  
**Format:** Interactive workshop with live demos and Q&A

---

## Session Structure

### **Hour 1: Container and Kubernetes Fundamentals (60 minutes)**

#### **1.1 Introduction & Welcome (10 minutes)**
- Workshop objectives and learning outcomes
- Participant introductions and experience levels
- Demo environment overview
  - Azure CLI setup
  - kubectl configuration
  - Docker Desktop for local examples
  - VS Code with Kubernetes extension

#### **1.2 Container Fundamentals (25 minutes)**
- **What are Containers?** (10 minutes)
  - Containerization vs Virtualization
  - Container benefits: portability, consistency, efficiency
  - Container lifecycle and images
  
- **Docker Basics** (15 minutes)
  - Dockerfile structure and best practices
  - Building and running containers
  - Container registries (ACR introduction)
  - **Demo 1.1:** Live container creation and deployment (10 minutes)

#### **1.3 Kubernetes Fundamentals (25 minutes)**
- **Why Kubernetes?** (10 minutes)
  - Container orchestration challenges
  - Kubernetes architecture overview
  - Master vs Worker nodes
  
- **Core Kubernetes Concepts** (15 minutes)
  - Pods, Services, Deployments
  - Namespaces and Labels
  - ConfigMaps and Secrets
  - **Demo 1.2:** kubectl commands and resource management

---

### **Hour 2: Azure Kubernetes Service (AKS) Deep Dive (60 minutes)**

#### **2.1 AKS Introduction and Architecture (20 minutes)**
- **What is AKS?** (10 minutes)
  - Managed Kubernetes service benefits
  - AKS vs self-managed Kubernetes
  - Azure integration advantages
  
- **AKS Architecture Components** (10 minutes)
  - Control plane (managed by Azure)
  - Node pools and scaling
  - Azure integrations (Microsoft Entra ID, ACR, Key Vault)
  - Networking models (kubenet vs Azure CNI)

#### **2.2 Creating and Managing AKS Clusters (25 minutes)**
- **Cluster Creation Options** (10 minutes)
  - Azure Portal vs Azure CLI vs ARM/Bicep templates
  - Cluster configuration considerations
  - Node pool configurations
  
- **Demo 2.1: AKS Cluster Creation and Management** (15 minutes)
  - Live demonstration of cluster creation process
  - Exploring cluster configuration options
  - Node pool management and scaling
  - Connecting to the cluster with kubectl
  ```bash
  # Demo commands - AKS cluster creation
  az group create --name aks-demo-rg --location eastus
  
  az aks create \
    --resource-group aks-demo-rg \
    --name aks-demo-cluster \
    --node-count 2 \
    --enable-addons monitoring \
    --generate-ssh-keys
  
  az aks get-credentials --resource-group aks-demo-rg --name aks-demo-cluster
  
  kubectl get nodes
  kubectl get pods --all-namespaces
  ```

#### **2.3 AKS Security and Best Practices (15 minutes)**
- **Security Considerations** (8 minutes)
  - Microsoft Entra ID integration
  - RBAC (Role-Based Access Control)
  - Network security groups and policies
  - Pod security standards
  
- **Best Practices** (7 minutes)
  - Resource quotas and limits
  - Monitoring and logging with Azure Monitor
  - Backup and disaster recovery strategies

---

### **Hour 3: Deployment Strategies and AKS on Azure Local (60 minutes)**

#### **3.1 Application Deployment on AKS (25 minutes)**
- **Deployment Strategies** (10 minutes)
  - Rolling updates
  - Blue-green deployments
  - Canary deployments
  - GitOps with AKS
  
- **Demo 3.1: Application Deployment to AKS** (15 minutes)
  - Live deployment demonstration
  - Service exposure and ingress configuration
  - Scaling and updating applications
  - Monitoring deployment status
  ```yaml
  # Demo deployment manifest
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: demo-app
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: demo-app
    template:
      metadata:
        labels:
          app: demo-app
      spec:
        containers:
        - name: app
          image: nginx:latest
          ports:
          - containerPort: 80
  ```

#### **3.2 AKS on Azure Local Fundamentals (30 minutes)**
- **Introduction to AKS on Azure Local** (15 minutes)
  - What is Azure Local? (formerly Azure Stack HCI)
  - Hybrid cloud scenarios and use cases
  - Edge computing requirements
  - Data sovereignty and compliance needs
  
- **AKS on Azure Local Architecture** (15 minutes)
  - Azure Local cluster requirements
  - AKS integration with Azure Local
  - Management from Azure Portal
  - Networking considerations for edge scenarios
  - **Demo 3.2:** AKS on Azure Local portal walkthrough and architecture review

#### **3.3 Deployment Considerations and Comparison (5 minutes)**
- **AKS vs AKS on Azure Local** (5 minutes)
  - When to use cloud vs edge
  - Performance and latency considerations
  - Cost implications
  - Management overhead comparison

---

## **Workshop Wrap-up and Q&A (10 minutes)**

### **Key Takeaways**
- Container fundamentals and Kubernetes basics
- AKS managed service benefits and architecture
- Security and best practices for production workloads
- AKS on Azure Local for edge and hybrid scenarios
- Next steps for continued learning

### **Additional Resources**
- [AKS Documentation](https://docs.microsoft.com/en-us/azure/aks/)
- [AKS Best Practices](https://docs.microsoft.com/en-us/azure/aks/best-practices)
- [Azure Local Documentation](https://docs.microsoft.com/en-us/azure-stack/hci/)
- [Kubernetes Learning Path](https://kubernetes.io/docs/concepts/)

### **Next Steps**
- Demo materials and scripts for self-practice
- AKS certification paths (AZ-104, AZ-204, AZ-305)
- Advanced topics: Service Mesh, GitOps, Advanced Security
- Follow-up hands-on workshop recommendations

---

*This workshop agenda is designed to provide comprehensive AKS knowledge through live demonstrations while covering both cloud and edge deployment scenarios. Timing can be adjusted based on audience engagement and questions during demos.*