# Demo 3.1: Application Deployment to AKS (15 minutes)

## Overview
This comprehensive demo demonstrates how to deploy applications to Azure Kubernetes Service (AKS), including service exposure, ingress configuration, scaling, and monitoring. Participants will learn end-to-end application deployment strategies.

## Prerequisites
- AKS cluster running and accessible
- kubectl configured for the cluster
- Docker image registry access (Azure Container Registry recommended)
- Basic understanding of Kubernetes resources

## Learning Objectives
- Deploy multi-tier applications to AKS
- Configure services and ingress for external access
- Implement horizontal pod autoscaling
- Monitor application health and performance
- Understand deployment strategies (rolling updates, blue-green)

---

## Step-by-Step Instructions

### Step 1: Prepare Application Images (2 minutes)

1. **Create a simple web application image (if not using existing):**
   ```bash
   # Create application directory
   mkdir aks-app-demo && cd aks-app-demo
   
   # Create a simple Node.js application
   cat > app.js << EOF
   const express = require('express');
   const os = require('os');
   const app = express();
   const port = 3000;
   
   app.get('/', (req, res) => {
     res.json({
       message: 'Hello from AKS!',
       hostname: os.hostname(),
       timestamp: new Date().toISOString(),
       version: 'v1.0'
     });
   });
   
   app.get('/health', (req, res) => {
     res.status(200).json({ status: 'healthy' });
   });
   
   app.listen(port, () => {
     console.log(\`App listening at http://localhost:\${port}\`);
   });
   EOF
   
   # Create package.json
   cat > package.json << EOF
   {
     "name": "aks-demo-app",
     "version": "1.0.0",
     "description": "Demo app for AKS deployment",
     "main": "app.js",
     "scripts": {
       "start": "node app.js"
     },
     "dependencies": {
       "express": "^4.18.0"
     }
   }
   EOF
   
   # Create Dockerfile
   cat > Dockerfile << EOF
   FROM node:18-alpine
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   EXPOSE 3000
   CMD ["npm", "start"]
   EOF
   ```

2. **Build and push image to Azure Container Registry (if available):**
   ```bash
   # Alternative: Use pre-built public images for the demo
   # We'll use nginx and a demo application image in the manifests
   echo "Using pre-built demo images for this workshop"
   ```

### Step 2: Create Namespace and Basic Deployment (3 minutes)

1. **Create a dedicated namespace:**
   ```bash
   kubectl create namespace aks-app-demo
   kubectl config set-context --current --namespace=aks-app-demo
   ```

2. **Create a basic deployment manifest:**
   ```bash
   cat > deployment.yaml << EOF
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: demo-app
     namespace: aks-app-demo
     labels:
       app: demo-app
       version: v1
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: demo-app
         version: v1
     template:
       metadata:
         labels:
           app: demo-app
           version: v1
       spec:
         containers:
         - name: app
           image: nginx:1.21-alpine
           ports:
           - containerPort: 80
           resources:
             requests:
               memory: "64Mi"
               cpu: "250m"
             limits:
               memory: "128Mi"
               cpu: "500m"
           livenessProbe:
             httpGet:
               path: /
               port: 80
             initialDelaySeconds: 30
             periodSeconds: 10
           readinessProbe:
             httpGet:
               path: /
               port: 80
             initialDelaySeconds: 5
             periodSeconds: 5
   EOF
   ```

3. **Deploy the application:**
   ```bash
   kubectl apply -f deployment.yaml
   ```

4. **Verify deployment:**
   ```bash
   kubectl get deployments
   kubectl get pods -o wide
   kubectl describe deployment demo-app
   ```

### Step 3: Create Services for Internal Communication (2 minutes)

1. **Create a ClusterIP service:**
   ```bash
   cat > service-clusterip.yaml << EOF
   apiVersion: v1
   kind: Service
   metadata:
     name: demo-app-service
     namespace: aks-app-demo
     labels:
       app: demo-app
   spec:
     type: ClusterIP
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
       name: http
     selector:
       app: demo-app
   EOF
   
   kubectl apply -f service-clusterip.yaml
   ```

2. **Test internal service connectivity:**
   ```bash
   kubectl get services
   kubectl describe service demo-app-service
   
   # Test from within cluster
   kubectl run test-pod --image=curlimages/curl:latest --rm -it --restart=Never -- curl http://demo-app-service.aks-app-demo.svc.cluster.local
   ```

### Step 4: External Access with LoadBalancer Service (2 minutes)

1. **Create a LoadBalancer service:**
   ```bash
   cat > service-loadbalancer.yaml << EOF
   apiVersion: v1
   kind: Service
   metadata:
     name: demo-app-lb
     namespace: aks-app-demo
     annotations:
       service.beta.kubernetes.io/azure-load-balancer-health-probe-request-path: /
   spec:
     type: LoadBalancer
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
       name: http
     selector:
       app: demo-app
   EOF
   
   kubectl apply -f service-loadbalancer.yaml
   ```

2. **Wait for external IP assignment:**
   ```bash
   kubectl get service demo-app-lb --watch
   # Press Ctrl+C when EXTERNAL-IP is assigned
   
   # Get the external IP
   EXTERNAL_IP=$(kubectl get service demo-app-lb -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
   echo "Application accessible at: http://$EXTERNAL_IP"
   ```

3. **Test external access:**
   ```bash
   curl http://$EXTERNAL_IP
   # Or open in browser
   ```

### Step 5: Configure Ingress Controller (3 minutes)

1. **Install NGINX Ingress Controller using Helm:**
   ```bash
   # Add Helm repository
   helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
   helm repo update
   
   # Install NGINX Ingress Controller
   helm install nginx-ingress ingress-nginx/ingress-nginx \
     --namespace ingress-nginx \
     --create-namespace \
     --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"=/healthz
   ```

2. **Wait for ingress controller to be ready:**
   ```bash
   kubectl get service -n ingress-nginx --watch
   # Press Ctrl+C when external IP is assigned
   
   INGRESS_IP=$(kubectl get service nginx-ingress-ingress-nginx-controller -n ingress-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
   echo "Ingress IP: $INGRESS_IP"
   ```

3. **Create an Ingress resource:**
   ```bash
   cat > ingress.yaml << EOF
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: demo-app-ingress
     namespace: aks-app-demo
     annotations:
       nginx.ingress.kubernetes.io/rewrite-target: /
   spec:
     ingressClassName: nginx
     rules:
     - host: demo-app.local
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: demo-app-service
               port:
                 number: 80
     - http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: demo-app-service
               port:
                 number: 80
   EOF
   
   kubectl apply -f ingress.yaml
   ```

4. **Test ingress access:**
   ```bash
   kubectl get ingress
   curl -H "Host: demo-app.local" http://$INGRESS_IP
   # Or test with IP directly
   curl http://$INGRESS_IP
   ```

### Step 6: Implement Horizontal Pod Autoscaler (2 minutes)

1. **Install metrics server (if not already present):**
   ```bash
   kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
   ```

2. **Create Horizontal Pod Autoscaler:**
   ```bash
   cat > hpa.yaml << EOF
   apiVersion: autoscaling/v2
   kind: HorizontalPodAutoscaler
   metadata:
     name: demo-app-hpa
     namespace: aks-app-demo
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: demo-app
     minReplicas: 3
     maxReplicas: 10
     metrics:
     - type: Resource
       resource:
         name: cpu
         target:
           type: Utilization
           averageUtilization: 70
     - type: Resource
       resource:
         name: memory
         target:
           type: Utilization
           averageUtilization: 80
   EOF
   
   kubectl apply -f hpa.yaml
   ```

3. **Monitor HPA status:**
   ```bash
   kubectl get hpa
   kubectl describe hpa demo-app-hpa
   ```

### Step 7: Rolling Updates and Deployment Strategies (2 minutes)

1. **Update application with rolling update:**
   ```bash
   # Update the image version
   kubectl set image deployment/demo-app app=nginx:1.22-alpine
   ```

2. **Monitor rolling update:**
   ```bash
   kubectl rollout status deployment/demo-app
   kubectl get pods -w
   # Press Ctrl+C to stop watching
   ```

3. **View rollout history:**
   ```bash
   kubectl rollout history deployment/demo-app
   ```

4. **Rollback if needed (demonstration):**
   ```bash
   # Rollback to previous version
   kubectl rollout undo deployment/demo-app
   kubectl rollout status deployment/demo-app
   ```

5. **Configure deployment strategy:**
   ```bash
   cat > deployment-strategy.yaml << EOF
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: demo-app-strategy
     namespace: aks-app-demo
   spec:
     replicas: 4
     strategy:
       type: RollingUpdate
       rollingUpdate:
         maxUnavailable: 1
         maxSurge: 1
     selector:
       matchLabels:
         app: demo-app-strategy
     template:
       metadata:
         labels:
           app: demo-app-strategy
       spec:
         containers:
         - name: app
           image: nginx:1.21-alpine
           ports:
           - containerPort: 80
   EOF
   
   kubectl apply -f deployment-strategy.yaml
   ```

### Step 8: Application Monitoring and Observability (1 minute)

1. **Check application logs:**
   ```bash
   kubectl logs -l app=demo-app --tail=10
   kubectl logs deployment/demo-app --follow
   # Press Ctrl+C to stop following
   ```

2. **Monitor resource usage:**
   ```bash
   kubectl top pods
   kubectl top nodes
   ```

3. **Check application health:**
   ```bash
   kubectl get pods -l app=demo-app
   kubectl describe pods -l app=demo-app
   ```

4. **View events:**
   ```bash
   kubectl get events --sort-by='.lastTimestamp' -n aks-app-demo
   ```

---

## Deployment Strategies Comparison

### Rolling Update (Default)
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 25%
    maxSurge: 25%
```
- **Pros**: Zero downtime, gradual rollout
- **Cons**: Mixed versions during update
- **Use case**: Most production deployments

### Recreate Strategy
```yaml
strategy:
  type: Recreate
```
- **Pros**: Simple, no version mixing
- **Cons**: Temporary downtime
- **Use case**: Development environments

### Blue-Green Deployment (Manual)
```bash
# Deploy new version with different label
kubectl apply -f deployment-v2.yaml
# Switch service selector after verification
kubectl patch service demo-app-service -p '{"spec":{"selector":{"version":"v2"}}}'
```

### Canary Deployment
```yaml
# Deploy canary with fewer replicas
# Use ingress or service mesh for traffic splitting
```

---

## Application Configuration Best Practices

### Resource Management
```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

### Health Checks
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 3000
  initialDelaySeconds: 5
  periodSeconds: 5
```

### Security Context
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1001
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
```

---

## Monitoring and Troubleshooting Commands

### Application Logs
```bash
kubectl logs <pod-name> -f
kubectl logs deployment/<deployment-name> --tail=100
kubectl logs -l app=<app-label> --previous
```

### Resource Usage
```bash
kubectl top pods
kubectl describe pod <pod-name>
kubectl get events --field-selector involvedObject.name=<pod-name>
```

### Network Connectivity
```bash
kubectl port-forward pod/<pod-name> 8080:80
kubectl exec -it <pod-name> -- nslookup <service-name>
kubectl get endpoints
```

### Scaling and Performance
```bash
kubectl get hpa
kubectl describe hpa <hpa-name>
kubectl scale deployment <deployment-name> --replicas=5
```

---

## Key Takeaways

1. **Kubernetes deployments provide declarative application management** with built-in rollback capabilities
2. **Service types determine network accessibility** - ClusterIP for internal, LoadBalancer for external access
3. **Ingress controllers enable advanced traffic management** and SSL termination
4. **Horizontal Pod Autoscaler automatically scales applications** based on resource metrics
5. **Health checks and resource limits are essential** for reliable applications

## Discussion Points

- **Service vs Ingress**: When to use each approach for external access
- **Resource sizing**: How to determine appropriate CPU and memory limits
- **Deployment strategies**: Choosing the right strategy for different scenarios
- **Monitoring integration**: Connecting with Azure Monitor and Application Insights

---

## Next Steps
- Explore advanced deployment patterns with service mesh (Istio/Linkerd)
- Implement GitOps workflows with ArgoCD or Flux
- Learn about stateful applications and persistent volumes
- Integrate with Azure services (Key Vault, Service Bus, etc.)

## Cleanup Commands

```bash
# Delete all resources in namespace
kubectl delete namespace aks-app-demo

# Remove ingress controller
helm uninstall nginx-ingress -n ingress-nginx
kubectl delete namespace ingress-nginx

# Reset default namespace
kubectl config set-context --current --namespace=default
```

## Troubleshooting

**Common Issues:**
- **Image pull errors**: Check registry authentication and image names
- **Service connectivity**: Verify selectors match pod labels
- **Ingress not working**: Check ingress controller installation and DNS
- **HPA not scaling**: Verify metrics server and resource requests are set

**Debug commands:**
```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name> --previous
kubectl get events --sort-by='.lastTimestamp'
kubectl port-forward <pod-name> <local-port>:<container-port>
```