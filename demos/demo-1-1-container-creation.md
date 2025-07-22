# Demo 1.1: Live Container Creation and Deployment (10 minutes)

## Overview
This demo demonstrates the fundamentals of container creation, building, and deployment using Docker. Participants will learn how to create a simple containerized application from scratch.

## Prerequisites
- Docker Desktop installed and running
- Basic command line knowledge
- Text editor (VS Code recommended)

## Learning Objectives
- Understand Dockerfile structure and syntax
- Build a custom container image
- Run containers locally
- Push images to a container registry

---

## Step-by-Step Instructions

### Step 1: Create a Simple Web Application (2 minutes)

1. **Create a new directory for the demo:**
   ```bash
   mkdir aks-demo-app
   cd aks-demo-app
   ```

2. **Create a simple HTML file (`index.html`):**
   ```bash
   cat > index.html << EOF
   <!DOCTYPE html>
   <html>
   <head>
       <title>AKS Demo App</title>
       <style>
           body { font-family: Arial, sans-serif; margin: 40px; background-color: #f0f8ff; }
           .container { max-width: 600px; margin: 0 auto; text-align: center; }
           h1 { color: #0078d4; }
       </style>
   </head>
   <body>
       <div class="container">
           <h1>Welcome to AKS Workshop!</h1>
           <p>This is a containerized web application running in Docker.</p>
           <p>Container ID: <span id="hostname"></span></p>
           <script>
               document.getElementById('hostname').textContent = window.location.hostname;
           </script>
       </div>
   </body>
   </html>
   EOF
   ```

### Step 2: Create a Dockerfile (2 minutes)

1. **Create a Dockerfile:**
   ```bash
   cat > Dockerfile << EOF
   # Use the official nginx base image
   FROM nginx:alpine

   # Copy our custom HTML file to the nginx html directory
   COPY index.html /usr/share/nginx/html/

   # Expose port 80
   EXPOSE 80

   # The default nginx command will start automatically
   CMD ["nginx", "-g", "daemon off;"]
   EOF
   ```

2. **Explain the Dockerfile components:**
   - `FROM`: Specifies the base image (nginx:alpine for small footprint)
   - `COPY`: Copies files from host to container
   - `EXPOSE`: Documents which port the container listens on
   - `CMD`: Defines the default command to run

### Step 3: Build the Container Image (2 minutes)

1. **Build the Docker image:**
   ```bash
   docker build -t aks-demo-app:v1.0 .
   ```

2. **Verify the image was created:**
   ```bash
   docker images | grep aks-demo-app
   ```

3. **Explain the build process:**
   - Docker reads the Dockerfile
   - Each instruction creates a new layer
   - Layers are cached for faster subsequent builds

### Step 4: Run the Container Locally (2 minutes)

1. **Run the container:**
   ```bash
   docker run -d -p 8080:80 --name aks-demo-container aks-demo-app:v1.0
   ```

2. **Verify the container is running:**
   ```bash
   docker ps
   ```

3. **Test the application:**
   ```bash
   # Open in browser or use curl
   curl http://localhost:8080
   ```

4. **View container logs:**
   ```bash
   docker logs aks-demo-container
   ```

### Step 5: Container Management Commands (2 minutes)

1. **Inspect the running container:**
   ```bash
   docker inspect aks-demo-container
   ```

2. **Execute commands inside the container:**
   ```bash
   docker exec -it aks-demo-container sh
   # Inside container:
   ls /usr/share/nginx/html/
   exit
   ```

3. **Stop and remove the container:**
   ```bash
   docker stop aks-demo-container
   docker rm aks-demo-container
   ```

4. **Clean up the image (optional):**
   ```bash
   docker rmi aks-demo-app:v1.0
   ```

---

## Key Takeaways

1. **Containers are lightweight and portable** - The same image runs consistently across environments
2. **Dockerfile defines the container blueprint** - Reproducible image creation
3. **Layered architecture** - Efficient storage and distribution
4. **Container lifecycle management** - Build, run, stop, remove operations

## Discussion Points

- **Containerization Benefits**: Consistency, portability, and isolation
- **Image vs Container**: Image is the template, container is the running instance
- **Best Practices**: Use minimal base images, avoid running as root, optimize for caching

---

## Next Steps
- In Demo 1.2, we'll explore Kubernetes resource management
- In Demo 2.1, we'll deploy this container to AKS
- Consider exploring multi-stage builds for production applications

## Troubleshooting

**Common Issues:**
- **Docker not running**: Ensure Docker Desktop is started
- **Port conflicts**: Use different port mapping if 8080 is in use
- **Permission errors**: Ensure user has Docker privileges

**Commands to verify setup:**
```bash
docker --version
docker info
docker run hello-world
```