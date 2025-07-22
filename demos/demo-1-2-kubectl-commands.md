# Demo 1.2: kubectl Commands and Resource Management

## Overview
This demo introduces participants to kubectl, the Kubernetes command-line tool, and demonstrates essential Kubernetes resource management operations. Participants will learn to interact with Kubernetes clusters and manage core resources.

## Prerequisites
- kubectl installed and configured
- Access to a Kubernetes cluster (local or cloud-based)
- Basic understanding of Kubernetes concepts

## Learning Objectives
- Master essential kubectl commands
- Understand Kubernetes resource types and their relationships
- Learn how to inspect and debug cluster resources
- Practice resource lifecycle management

---

## Step-by-Step Instructions

### Step 1: Cluster Connection and Verification (2 minutes)

1. **Verify kubectl installation and cluster connection:**
   ```bash
   kubectl version --client --short
   kubectl cluster-info
   ```

2. **Check cluster nodes:**
   ```bash
   kubectl get nodes
   kubectl get nodes -o wide
   ```

3. **View cluster configuration:**
   ```bash
   kubectl config current-context
   kubectl config get-contexts
   ```

### Step 2: Exploring Namespaces (3 minutes)

1. **List all namespaces:**
   ```bash
   kubectl get namespaces
   # or short form
   kubectl get ns
   ```

2. **Create a demo namespace:**
   ```bash
   kubectl create namespace aks-demo
   ```

3. **Set namespace as default for current context:**
   ```bash
   kubectl config set-context --current --namespace=aks-demo
   ```

4. **Verify namespace switch:**
   ```bash
   kubectl config view --minify | grep namespace
   ```

### Step 3: Pod Management (5 minutes)

1. **Create a simple pod using imperative command:**
   ```bash
   kubectl run demo-pod --image=nginx:latest --port=80
   ```

2. **List pods in current namespace:**
   ```bash
   kubectl get pods
   kubectl get pods -o wide
   ```

3. **Describe the pod (detailed information):**
   ```bash
   kubectl describe pod demo-pod
   ```

4. **View pod logs:**
   ```bash
   kubectl logs demo-pod
   ```

5. **Execute commands inside the pod:**
   ```bash
   kubectl exec -it demo-pod -- sh
   # Inside pod:
   ls /usr/share/nginx/html/
   whoami
   exit
   ```

6. **Create a pod with a custom YAML manifest:**
   ```bash
   cat > demo-pod-advanced.yaml << EOF
   apiVersion: v1
   kind: Pod
   metadata:
     name: demo-pod-advanced
     namespace: aks-demo
     labels:
       app: demo
       version: v1
   spec:
     containers:
     - name: nginx
       image: nginx:1.21
       ports:
       - containerPort: 80
       resources:
         requests:
           memory: "64Mi"
           cpu: "250m"
         limits:
           memory: "128Mi"
           cpu: "500m"
   EOF
   ```

7. **Apply the YAML manifest:**
   ```bash
   kubectl apply -f demo-pod-advanced.yaml
   ```

### Step 4: Service Management (3 minutes)

1. **Create a service to expose the pod:**
   ```bash
   kubectl expose pod demo-pod --port=80 --target-port=80 --name=demo-service
   ```

2. **List services:**
   ```bash
   kubectl get services
   kubectl get svc -o wide
   ```

3. **Describe the service:**
   ```bash
   kubectl describe service demo-service
   ```

4. **Create a service using YAML:**
   ```bash
   cat > demo-service.yaml << EOF
   apiVersion: v1
   kind: Service
   metadata:
     name: demo-service-yaml
     namespace: aks-demo
   spec:
     selector:
       app: demo
     ports:
       - protocol: TCP
         port: 80
         targetPort: 80
     type: ClusterIP
   EOF
   
   kubectl apply -f demo-service.yaml
   ```

### Step 5: Deployment Management (4 minutes)

1. **Create a deployment:**
   ```bash
   kubectl create deployment demo-deployment --image=nginx:latest --replicas=3
   ```

2. **View deployment status:**
   ```bash
   kubectl get deployments
   kubectl get deploy -o wide
   ```

3. **View replica sets:**
   ```bash
   kubectl get replicasets
   kubectl get rs
   ```

4. **Scale the deployment:**
   ```bash
   kubectl scale deployment demo-deployment --replicas=5
   ```

5. **Watch the scaling in real-time:**
   ```bash
   kubectl get pods -w
   # Press Ctrl+C to stop watching
   ```

6. **Update deployment image:**
   ```bash
   kubectl set image deployment/demo-deployment nginx=nginx:1.21
   ```

7. **View rollout status:**
   ```bash
   kubectl rollout status deployment/demo-deployment
   ```

### Step 6: Resource Inspection and Debugging (3 minutes)

1. **Get all resources in namespace:**
   ```bash
   kubectl get all
   ```

2. **Use labels to filter resources:**
   ```bash
   kubectl get pods -l app=demo
   kubectl get all -l app=demo-deployment
   ```

3. **View resource usage:**
   ```bash
   kubectl top nodes
   kubectl top pods
   ```

4. **Get resource definitions in YAML:**
   ```bash
   kubectl get pod demo-pod -o yaml
   kubectl get service demo-service -o json
   ```

5. **Edit resources on the fly:**
   ```bash
   kubectl edit deployment demo-deployment
   # This opens the resource in your default editor
   ```

### Step 7: Cleanup and Resource Deletion (2 minutes)

1. **Delete individual resources:**
   ```bash
   kubectl delete pod demo-pod
   kubectl delete pod demo-pod-advanced
   kubectl delete service demo-service
   kubectl delete service demo-service-yaml
   kubectl delete deployment demo-deployment
   ```

2. **Delete using manifest files:**
   ```bash
   kubectl delete -f demo-pod-advanced.yaml
   kubectl delete -f demo-service.yaml
   ```

3. **Delete all resources by label:**
   ```bash
   kubectl delete all -l app=demo
   ```

4. **Delete the demo namespace (and all resources within):**
   ```bash
   kubectl delete namespace aks-demo
   ```

5. **Reset namespace to default:**
   ```bash
   kubectl config set-context --current --namespace=default
   ```

---

## Essential kubectl Commands Reference

### Resource Management
```bash
# Create resources
kubectl create <resource-type> <name> [options]
kubectl apply -f <file.yaml>

# View resources
kubectl get <resource-type>
kubectl describe <resource-type> <name>

# Update resources
kubectl edit <resource-type> <name>
kubectl patch <resource-type> <name> -p '<patch>'

# Delete resources
kubectl delete <resource-type> <name>
kubectl delete -f <file.yaml>
```

### Debugging Commands
```bash
# Logs
kubectl logs <pod-name>
kubectl logs -f <pod-name>  # Follow logs

# Execute commands
kubectl exec -it <pod-name> -- <command>

# Port forwarding
kubectl port-forward <pod-name> <local-port>:<pod-port>

# Resource usage
kubectl top nodes
kubectl top pods
```

### Context and Configuration
```bash
# Contexts
kubectl config get-contexts
kubectl config use-context <context-name>

# Namespaces
kubectl config set-context --current --namespace=<namespace>

# Cluster info
kubectl cluster-info
kubectl version
```

---

## Key Takeaways

1. **kubectl is the primary interface** for interacting with Kubernetes clusters
2. **Declarative management** with YAML is preferred for production
3. **Labels and selectors** are crucial for resource organization and selection
4. **Namespaces** provide logical isolation within clusters
5. **Resource relationships** - Deployments manage ReplicaSets, which manage Pods

## Discussion Points

- **Imperative vs Declarative**: When to use each approach
- **Resource Hierarchy**: Understanding owner-references
- **Troubleshooting Strategy**: Logs, describe, and events
- **Production Best Practices**: Version control, GitOps, resource limits

---

## Next Steps
- In Demo 2.1, we'll create and manage an AKS cluster
- In Demo 3.1, we'll deploy applications to AKS using these kubectl skills
- Explore advanced kubectl plugins and tools (k9s, kubectx, etc.)

## Troubleshooting

**Common Issues:**
- **No cluster connection**: Check kubectl config and cluster availability
- **Permission errors**: Verify RBAC permissions for your user/service account
- **Resource not found**: Ensure correct namespace context

**Useful debugging commands:**
```bash
kubectl cluster-info dump
kubectl get events --sort-by='.lastTimestamp'
kubectl api-resources
kubectl explain <resource-type>
```