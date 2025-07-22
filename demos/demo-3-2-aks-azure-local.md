# Demo 3.2: AKS on Azure Local Portal Walkthrough and Architecture Review

## Overview
This demo provides a comprehensive walkthrough of AKS on Azure Local (formerly Azure Stack HCI), covering architecture, use cases, and management through the Azure Portal. Participants will understand the hybrid cloud deployment model and edge computing capabilities.

## Prerequisites
- Access to Azure Portal
- Understanding of hybrid cloud concepts
- Basic knowledge of Azure Local (Azure Stack HCI)
- Familiarity with AKS concepts from previous demos

## Learning Objectives
- Understand AKS on Azure Local architecture and components
- Identify use cases for edge and hybrid deployments
- Navigate the Azure Portal for AKS on Azure Local management
- Compare cloud vs edge deployment considerations
- Learn about data sovereignty and compliance requirements

---

## Step-by-Step Instructions

### Step 1: Introduction to Azure Local and AKS Integration (5 minutes)

#### **What is Azure Local?**

1. **Azure Local Overview:**
   - Formerly known as Azure Stack HCI
   - Hybrid cloud infrastructure platform
   - Runs on-premises with Azure management
   - Provides cloud services at the edge

2. **Key Components:**
   - Hyper-converged infrastructure (HCI)
   - Software-defined networking and storage
   - Azure Arc integration
   - Azure Resource Manager extensions

#### **AKS on Azure Local Benefits:**

1. **Edge Computing Capabilities:**
   - Low latency for edge applications
   - Local data processing and storage
   - Reduced bandwidth requirements
   - Offline operation capabilities

2. **Regulatory Compliance:**
   - Data sovereignty requirements
   - Local data residency
   - Industry-specific regulations
   - Air-gapped environments

3. **Hybrid Operations:**
   - Consistent API and tooling
   - Unified management from Azure
   - Seamless application portability
   - Cloud-native development practices

### Step 2: Architecture Deep Dive (8 minutes)

#### **AKS on Azure Local Architecture Components:**

1. **Infrastructure Layer:**
   ```
   ┌─────────────────────────────────────────────────────┐
   │                Azure Portal                         │
   │            (Management Plane)                       │
   └─────────────────┬───────────────────────────────────┘
                     │ Azure Arc Connection
   ┌─────────────────▼───────────────────────────────────┐
   │              Azure Local                            │
   │  ┌─────────────────┐  ┌─────────────────────────┐   │
   │  │   AKS Cluster   │  │    Host Management      │   │
   │  │   (Kubernetes)  │  │    (Windows Admin       │   │
   │  │                 │  │     Center)             │   │
   │  └─────────────────┘  └─────────────────────────┘   │
   │  ┌─────────────────────────────────────────────┐   │
   │  │          Hyper-V Infrastructure             │   │
   │  │    (Compute, Storage, Networking)           │   │
   │  └─────────────────────────────────────────────┘   │
   │  ┌─────────────────────────────────────────────┐   │
   │  │         Physical Hardware                   │   │
   │  │    (Servers, Storage, Network)              │   │
   │  └─────────────────────────────────────────────┘   │
   └─────────────────────────────────────────────────────┘
   ```

2. **Network Architecture:**
   - Management network for Azure Local cluster
   - Kubernetes network for pod communications
   - External network for application access
   - Storage network for persistent volumes

3. **Control Plane:**
   - Managed through Azure Portal
   - Azure Arc-enabled Kubernetes
   - GitOps and policy management
   - Monitoring and logging integration

#### **Comparison: Cloud AKS vs AKS on Azure Local:**

| Aspect | Cloud AKS | AKS on Azure Local |
|--------|-----------|-------------------|
| **Location** | Azure Regions | On-premises/Edge |
| **Latency** | Internet-dependent | Local/Ultra-low |
| **Control Plane** | Fully managed | Arc-managed |
| **Data Residency** | Azure regions | Local control |
| **Connectivity** | Internet required | Can run offline |
| **Scaling** | Unlimited | Hardware limited |
| **Cost Model** | Pay-per-use | Infrastructure + licensing |

### Step 3: Azure Portal Navigation for AKS on Azure Local (7 minutes)

#### **Portal Walkthrough - Live Demonstration:**

1. **Navigate to Azure Arc in the Portal:**
   ```
   Azure Portal → All Services → Azure Arc → Kubernetes clusters
   ```

2. **AKS on Azure Local Dashboard:**
   - Cluster overview and status
   - Node information and health
   - Workload distribution
   - Resource utilization metrics

3. **Key Portal Sections to Explore:**

   **a) Cluster Configuration:**
   - Kubernetes version
   - Node pools configuration
   - Network settings
   - Security and authentication

   **b) Monitoring and Insights:**
   - Container Insights integration
   - Performance metrics
   - Log analytics workspace
   - Alerts and notifications

   **c) Applications and Workloads:**
   - Deployed applications view
   - Service and ingress configuration
   - Persistent volume claims
   - ConfigMaps and Secrets

   **d) GitOps (if enabled):**
   - Git repository connections
   - Configuration deployment status
   - Policy compliance
   - Automated sync status

#### **Configuration Examples in Portal:**

1. **Cluster Creation Workflow:**
   ```
   Create Resource → Kubernetes - Azure Arc → 
   Select "AKS on Azure Local" → 
   Configure basic settings → 
   Networking configuration → 
   Authentication → 
   Review and create
   ```

2. **Monitoring Setup:**
   ```
   Cluster → Monitoring → Insights → 
   Enable Container Insights → 
   Configure Log Analytics → 
   Set up alerts and dashboards
   ```

### Step 4: Use Cases and Deployment Scenarios (8 minutes)

#### **Primary Use Cases:**

1. **Manufacturing and Industrial IoT:**
   ```yaml
   # Example: Factory automation workload
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: factory-automation
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: factory-automation
     template:
       metadata:
         labels:
           app: factory-automation
       spec:
         containers:
         - name: control-system
           image: factory-control:latest
           resources:
             requests:
               memory: "256Mi"
               cpu: "500m"
           env:
           - name: PLC_ENDPOINT
             value: "192.168.1.100"
           - name: EDGE_MODE
             value: "true"
   ```

2. **Retail and Point of Sale:**
   - Real-time inventory management
   - Customer analytics at edge
   - Offline operation capability
   - Quick response times

3. **Healthcare and Life Sciences:**
   - Patient data processing
   - Medical imaging analysis
   - Regulatory compliance
   - Data sovereignty

4. **Financial Services:**
   - Branch banking applications
   - Risk analysis at edge
   - Regulatory requirements
   - High availability needs

#### **Architecture Patterns:**

1. **Hub and Spoke Model:**
   ```
   ┌─────────────────┐    ┌─────────────────┐
   │   Azure Cloud   │◄──►│  AKS on Azure   │
   │   (Central Hub) │    │     Local       │
   │                 │    │   (Edge Spoke)  │
   └─────────────────┘    └─────────────────┘
            │                        │
            ▼                        ▼
   ┌─────────────────┐    ┌─────────────────┐
   │ Data Analytics  │    │ Local Processing│
   │ Machine Learning│    │ Real-time Apps  │
   └─────────────────┘    └─────────────────┘
   ```

2. **Multi-Site Edge Deployment:**
   ```
   Azure Cloud Management
           │
       ┌───┴───┬───────┬───────┐
       │       │       │       │
   Site A   Site B   Site C   Site D
   (Store)  (Factory)(Branch) (Clinic)
   ```

### Step 5: Management and Operations (5 minutes)

#### **Day-to-Day Operations:**

1. **Application Deployment:**
   ```bash
   # Deploy applications using standard kubectl
   kubectl apply -f app-deployment.yaml
   
   # Or through GitOps configuration
   # Git repository → Azure Arc → AKS on Azure Local
   ```

2. **Monitoring and Alerting:**
   - Azure Monitor integration
   - Custom metrics collection
   - Alert rules configuration
   - Dashboard creation

3. **Security and Compliance:**
   - Azure Policy enforcement
   - Role-based access control
   - Network security policies
   - Compliance reporting

#### **Operational Considerations:**

1. **Backup and Disaster Recovery:**
   - Cluster configuration backup
   - Application data backup
   - Recovery procedures
   - Business continuity planning

2. **Updates and Maintenance:**
   - Kubernetes version updates
   - Azure Local infrastructure updates
   - Application rollouts
   - Maintenance windows

3. **Capacity Planning:**
   - Resource monitoring
   - Growth projections
   - Hardware planning
   - Performance optimization

### Step 6: Integration with Azure Services (7 minutes)

#### **Azure Arc Integration:**

1. **Arc-Enabled Kubernetes Features:**
   ```yaml
   # GitOps configuration example
   apiVersion: source.toolkit.fluxcd.io/v1beta2
   kind: GitRepository
   metadata:
     name: app-source
     namespace: flux-system
   spec:
     interval: 1m
     url: https://github.com/company/edge-apps
     ref:
       branch: main
   ```

2. **Policy and Governance:**
   - Azure Policy for Kubernetes
   - Compliance assessment
   - Security baselines
   - Configuration drift detection

#### **Hybrid Data Services:**

1. **Azure Arc-Enabled Data Services:**
   - SQL Managed Instance
   - PostgreSQL Hyperscale
   - Data processing at edge
   - Hybrid data scenarios

2. **Storage Integration:**
   - Azure Stack HCI storage
   - Persistent volume provisioning
   - Backup to Azure
   - Disaster recovery

#### **Security Integration:**

1. **Microsoft Entra ID Integration:**
   - Single sign-on
   - Conditional access
   - Multi-factor authentication
   - Audit logging

2. **Azure Security Center:**
   - Security recommendations
   - Threat detection
   - Vulnerability assessment
   - Compliance monitoring

### Step 7: Cost Considerations and ROI (3 minutes)

#### **Cost Components:**

1. **Infrastructure Costs:**
   - Hardware acquisition
   - Datacenter/facility costs
   - Power and cooling
   - Network connectivity

2. **Software Licensing:**
   - Windows Server Datacenter
   - Azure Local licensing
   - Management tools
   - Third-party software

3. **Operational Costs:**
   - IT staff and training
   - Maintenance contracts
   - Monitoring and management
   - Backup and recovery

#### **ROI Considerations:**

1. **Business Benefits:**
   - Reduced latency
   - Improved compliance
   - Enhanced security
   - Business continuity

2. **Technical Benefits:**
   - Consistent operations
   - Simplified management
   - Hybrid capabilities
   - Future-proof architecture

---

## Key Decision Factors

### When to Choose AKS on Azure Local:

✅ **Low latency requirements** (< 10ms)
✅ **Data sovereignty needs**
✅ **Regulatory compliance requirements**
✅ **Limited or unreliable internet connectivity**
✅ **Large amounts of local data processing**
✅ **Air-gapped environments**

### When to Choose Cloud AKS:

✅ **Global scale requirements**
✅ **Variable workload patterns**
✅ **Minimal infrastructure management**
✅ **Cost optimization priority**
✅ **Rapid development and testing**
✅ **Cloud-native applications**

---

## Best Practices Checklist

### Planning
- [ ] Assess latency requirements
- [ ] Evaluate compliance needs
- [ ] Plan network architecture
- [ ] Design disaster recovery
- [ ] Calculate total cost of ownership

### Implementation
- [ ] Follow Azure Local sizing guidelines
- [ ] Implement monitoring from day one
- [ ] Configure automated backups
- [ ] Set up security policies
- [ ] Plan for capacity growth

### Operations
- [ ] Establish operational procedures
- [ ] Train operations team
- [ ] Implement change management
- [ ] Monitor performance metrics
- [ ] Regular security assessments

---

## Key Takeaways

1. **AKS on Azure Local enables cloud-native applications at the edge** with consistent management
2. **Hybrid architecture provides flexibility** for different deployment requirements
3. **Azure Portal provides unified management** for both cloud and edge clusters
4. **Use cases focus on latency, compliance, and data sovereignty** requirements
5. **Investment in edge infrastructure requires careful ROI analysis**

## Discussion Points

- **Edge vs Cloud trade-offs**: Performance, cost, complexity
- **Hybrid application architecture**: Data flow and processing distribution
- **Operational challenges**: Skills, processes, and tooling
- **Future roadmap**: Integration with other Azure services

---

## Next Steps and Advanced Topics

### Immediate Next Steps:
1. **Evaluate specific use cases** in your organization
2. **Conduct proof of concept** for edge scenarios
3. **Assess infrastructure readiness** for Azure Local
4. **Plan pilot deployment** for low-risk workloads

### Advanced Learning Topics:
1. **Azure Arc-enabled services** deep dive
2. **GitOps implementation** for edge deployments
3. **Multi-cluster management** strategies
4. **Advanced networking** configurations
5. **Integration with Azure IoT** services

### Resources for Continued Learning:
- [Azure Local documentation](https://docs.microsoft.com/en-us/azure-stack/hci/)
- [AKS on Azure Local overview](https://docs.microsoft.com/en-us/azure/aks/hybrid/)
- [Azure Arc documentation](https://docs.microsoft.com/en-us/azure/azure-arc/)
- [Edge computing patterns](https://docs.microsoft.com/en-us/azure/architecture/example-scenario/hybrid/arc-hybrid-kubernetes)

---

## Summary

AKS on Azure Local represents the future of hybrid cloud computing, enabling organizations to run cloud-native applications where they need them most. Whether for compliance, performance, or business requirements, this platform provides the flexibility and consistency needed for modern edge computing scenarios.

The combination of Azure management capabilities with on-premises deployment provides the best of both worlds: cloud innovation with edge control.