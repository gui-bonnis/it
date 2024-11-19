How can I create a local development environment with microk8s to deploy java microservice with kubernetes after run mave deploy

###### ChatGPT said:

ChatGPT

To create a local development environment with MicroK8s for deploying a Java microservice after running Maven deploy, follow these steps:

### Step 1: Set Up MicroK8s

1. **Install MicroK8s**: Ensure you have MicroK8s installed on your local machine. On Ubuntu, you can install it with:
    
    bash
    
    Copy code
    
    `sudo snap install microk8s --classic`
    
2. **Enable Necessary Add-ons**: MicroK8s comes with several add-ons that you'll need, such as `dns`, `ingress`, and `storage`. Enable them using:
    
    bash
    
    Copy code
    
    `microk8s enable dns ingress storage`
    
    Optionally, enable **MetalLB** for load balancing:
    
    bash
    
    Copy code
    
    `microk8s enable metallb:192.168.15.100-192.168.15.200`
    
    Adjust the IP range according to your network.
    
3. **Create a Namespace**: Set up a namespace for your local development:
    
    bash
    
    Copy code
    
    `microk8s kubectl create namespace local-dev`
    

### Step 2: Prepare Your Java Microservice

1. **Create Spring Boot Application**: If you haven't already, you can scaffold a simple Spring Boot microservice using:
    
    bash
    
    Copy code
    
    `mvn archetype:generate -DgroupId=com.example -DartifactId=demo -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false`
    
2. **Containerize the Application**: Create a `Dockerfile` for your Spring Boot application. Here's a basic example:
    
    dockerfile
    
    Copy code
    
    `FROM openjdk:17-jdk-slim VOLUME /tmp ARG JAR_FILE=target/demo-0.0.1-SNAPSHOT.jar COPY ${JAR_FILE} app.jar ENTRYPOINT ["java","-jar","/app.jar"]`
    
3. **Build and Push the Docker Image**: After setting up the `Dockerfile`, build the image:
    
    bash
    
    Copy code
    
    `mvn clean package docker build -t localhost:32000/demo:latest .`
    
    Push the image to MicroK8s local registry:
    
    bash
    
    Copy code
    
    `microk8s enable registry docker tag demo:latest localhost:32000/demo:latest docker push localhost:32000/demo:latest`
    

### Step 3: Deploy to Kubernetes

1. **Create a Kubernetes Deployment File**: Create a `deployment.yaml` file for your microservice:
    
    yaml
    
    Copy code
    
    `apiVersion: apps/v1 kind: Deployment metadata:   name: demo   namespace: local-dev spec:   replicas: 1   selector:     matchLabels:       app: demo   template:     metadata:       labels:         app: demo     spec:       containers:       - name: demo         image: localhost:32000/demo:latest         ports:         - containerPort: 8080 --- apiVersion: v1 kind: Service metadata:   name: demo   namespace: local-dev spec:   type: NodePort   ports:   - port: 8080     targetPort: 8080     nodePort: 30001   selector:     app: demo`
    
2. **Apply the Deployment**: Deploy your application using `kubectl`:
    
    bash
    
    Copy code
    
    `microk8s kubectl apply -f deployment.yaml`
    
3. **Check the Status**: Verify that your microservice is running:
    
    bash
    
    Copy code
    
    `microk8s kubectl get pods -n local-dev`
    

### Step 4: Expose the Service

To access the service from your browser or Postman:

- If you’re using **MetalLB**, access it via the external IP.
    
    bash
    
    Copy code
    
    `microk8s kubectl get svc -n local-dev`
    
    Use the IP of the NodePort or LoadBalancer.
    
- If you’re using **Ingress**: Add an `Ingress` rule to route traffic:
    
    yaml
    
    Copy code
    
    `apiVersion: networking.k8s.io/v1 kind: Ingress metadata:   name: demo-ingress   namespace: local-dev   annotations:     nginx.ingress.kubernetes.io/rewrite-target: / spec:   rules:   - host: demo.local.dev     http:       paths:       - path: /         pathType: Prefix         backend:           service:             name: demo             port:               number: 8080`
    
    Update your `/etc/hosts` file:
    
    bash
    
    Copy code
    
    `sudo nano /etc/hosts`
    
    Add:
    
    lua
    
    Copy code
    
    `192.168.15.12 demo.local.dev`
    

### Step 5: Automate Deployment with Maven

1. **Maven `fabric8` Plugin**: Use the `fabric8` plugin to deploy your microservice to Kubernetes after running `mvn deploy`. Add this to your `pom.xml`:
    
    xml
    
    Copy code
    
    `<build>   <plugins>     <plugin>       <groupId>io.fabric8</groupId>       <artifactId>fabric8-maven-plugin</artifactId>       <version>7.3.1</version>       <executions>         <execution>           <goals>             <goal>deploy</goal>           </goals>         </execution>       </executions>     </plugin>   </plugins> </build>`
    
    Run Maven deploy:
    
    bash
    
    Copy code
    
    `mvn clean package fabric8:deploy`
    

### Conclusion

You now have a full local development environment using MicroK8s, and your Java microservice is set up to deploy to the local Kubernetes cluster. You can automate the deployment using Maven and further optimize your CI/CD pipeline.

4o