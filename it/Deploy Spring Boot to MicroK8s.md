How can I deploy docker app into kubernetes at microk8s at my ubuntu? Please consider a helloworld springboot microservices. Add the microservices code, the docker file, kubernetes manifest, microk8s registry, and all steps from create the jar until access the url in a browser

###### ChatGPT said:

ChatGPT

Memory updated

To deploy a Spring Boot "Hello World" microservice Docker app into MicroK8s on Ubuntu, you need to follow these steps:

### 1. Create the Spring Boot "Hello World" Microservice

#### **Step 1: Generate the Spring Boot App**

You can create a simple Spring Boot microservice using [Spring Initializr](https://start.spring.io/) or using the following Maven structure:

bash

Copy code

`mkdir helloworld-springboot cd helloworld-springboot`

Create the following files:

**`pom.xml`**:

xml

Copy code

`<project xmlns="http://maven.apache.org/POM/4.0.0"          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"          xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">     <modelVersion>4.0.0</modelVersion>     <groupId>com.example</groupId>     <artifactId>helloworld</artifactId>     <version>1.0.0</version>     <packaging>jar</packaging>      <parent>         <groupId>org.springframework.boot</groupId>         <artifactId>spring-boot-starter-parent</artifactId>         <version>3.3.1</version>         <relativePath/> <!-- lookup parent from repository -->     </parent>      <dependencies>         <dependency>             <groupId>org.springframework.boot</groupId>             <artifactId>spring-boot-starter-web</artifactId>         </dependency>     </dependencies>      <build>         <plugins>             <plugin>                 <groupId>org.springframework.boot</groupId>                 <artifactId>spring-boot-maven-plugin</artifactId>             </plugin>         </plugins>     </build> </project>`

**`src/main/java/com/example/helloworld/HelloworldApplication.java`**:

java

Copy code

`package com.example.helloworld;  import org.springframework.boot.SpringApplication; import org.springframework.boot.autoconfigure.SpringBootApplication; import org.springframework.web.bind.annotation.GetMapping; import org.springframework.web.bind.annotation.RestController;  @SpringBootApplication @RestController public class HelloworldApplication {      public static void main(String[] args) {         SpringApplication.run(HelloworldApplication.class, args);     }      @GetMapping("/")     public String helloWorld() {         return "Hello, World!";     } }`

#### **Step 2: Build the JAR**

Run the following command to build the Spring Boot app:

bash

Copy code

`mvn clean package`

### 2. Create Docker Image

#### **Step 1: Create a Dockerfile**

Create a file named `Dockerfile` in the root of your project:

Dockerfile

Copy code

`FROM eclipse-temurin:17-jdk-alpine VOLUME /tmp COPY target/helloworld-1.0.0.jar app.jar ENTRYPOINT ["java","-jar","/app.jar"]`

#### **Step 2: Build the Docker Image**

First, make sure Docker is installed on your machine.

Run the following command to build the Docker image:

bash

Copy code

`docker build -t helloworld-springboot .`

### 3. MicroK8s Setup

#### **Step 1: Install MicroK8s**

bash

Copy code

`sudo snap install microk8s --classic`

#### **Step 2: Enable Necessary Add-ons**

Enable DNS, the MicroK8s registry, and the ingress controller:

bash

Copy code

`microk8s enable dns registry ingress`

#### **Step 3: Verify MicroK8s**

Check the status of MicroK8s to ensure it's running:

bash

Copy code

`microk8s status`

If needed, join the MicroK8s group to avoid using `sudo`:

bash

Copy code

`sudo usermod -aG microk8s $USER sudo chown -f -R $USER ~/.kube`

Log out and log back in for the changes to take effect.

### 4. Push the Docker Image to MicroK8s Registry

Tag the Docker image for the local MicroK8s registry:

bash

Copy code

`docker tag helloworld-springboot localhost:32000/helloworld-springboot`

Push the image to the MicroK8s registry:

bash

Copy code

`docker push localhost:32000/helloworld-springboot`

### 5. Deploy to Kubernetes

#### **Step 1: Create Kubernetes Deployment and Service**

Create a file named `helloworld-deployment.yaml` with the following content:

yaml

Copy code

`apiVersion: apps/v1 kind: Deployment metadata:   name: helloworld-deployment spec:   replicas: 1   selector:     matchLabels:       app: helloworld   template:     metadata:       labels:         app: helloworld     spec:       containers:       - name: helloworld         image: localhost:32000/helloworld-springboot         ports:         - containerPort: 8080 --- apiVersion: v1 kind: Service metadata:   name: helloworld-service spec:   selector:     app: helloworld   ports:     - protocol: TCP       port: 80       targetPort: 8080   type: ClusterIP`

Apply the manifest to MicroK8s:

bash

Copy code

`microk8s kubectl apply -f helloworld-deployment.yaml`

#### **Step 2: Expose the Service via Ingress**

Create an ingress resource by adding a file `helloworld-ingress.yaml`:

yaml

Copy code

`apiVersion: networking.k8s.io/v1 kind: Ingress metadata:   name: helloworld-ingress   annotations:     nginx.ingress.kubernetes.io/rewrite-target: / spec:   rules:   - host: helloworld.local     http:       paths:       - path: /         pathType: Prefix         backend:           service:             name: helloworld-service             port:               number: 80`

Apply the ingress resource:

bash

Copy code

`microk8s kubectl apply -f helloworld-ingress.yaml`

#### **Step 3: Access the Application**

Add an entry to your `/etc/hosts` file to access the application locally:

bash

Copy code

`sudo nano /etc/hosts`

Add the following line:

text

Copy code

`127.0.0.1   helloworld.local`

Now, you can access your Spring Boot application by navigating to `http://helloworld.local` in your browser.

---

This step-by-step guide helps you deploy a Spring Boot "Hello World" microservice on MicroK8s in Ubuntu. If you'd like further customization or modifications, feel free to ask!