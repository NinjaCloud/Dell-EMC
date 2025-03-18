**Task: Deploy a Web Application Using Docker and Kubernetes**  

1. **Clone the GitHub Repository**  
   - Retrieve the repository from the following URL:  
     `https://github.com/NinjaCloud/tutum-hello-world-rebuild.git`  

2. **Modify Application Content**  
   - Navigate to the `www` directory and update its content as required.  

3. **Build and Push Docker Image**  
   - Build a Docker image named **`mywebapp:v1`**.  
   - Push the image to your **Docker Hub** registry.  

4. **Create a Kubernetes Deployment**  
   - Deploy the application using the Docker image from **Docker Hub**.  
   - Ensure the deployment has:  
     - **Deployment Name**: `mywebapp`  
     - **Replica Count**: 3  
     - **Container Name**: `mycon1`  
     - **Label**: `app=mywebapp`  

5. **Expose the Application Using a LoadBalancer Service**  
   - Create a **LoadBalancer** service to expose the deployment.  
   - The service should listen on **port 8085**.  

6. **Access the Application**  
   - Retrieve the external IP of the service.  
   - Open a browser and access the application via `http://<EXTERNAL-IP>:8085`.  

