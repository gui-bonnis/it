I would like your help to create a local development env with the following structure

/ local-dev-environment
  ├── docker-compose.yml        # Docker Compose file to run local services
  ├── k8s                       # Kubernetes manifests for local (MicroK8s) deployment
  │   ├── sonarqube.yaml        # ArgoCD, Vault, SonarQube, etc. manifests
  │   ├── argocd.yaml           # ArgoCD setup YAML
  │   └── keycloak.yaml            # Vault setup YAML
  ├── terraform                 # Terraform configurations for infrastructure provisioning
  │   ├── sonarqube.tf          # Terraform setup for SonarQube
  │   ├── keycloak.tf              # Terraform setup for keycloak
  │   └── argocd.tf             # Terraform setup for ArgoCD
  ├── scripts                   # Shell scripts or automation scripts to deploy and configure services
  │   ├── setup-sonar.sh        # Scripts for local setup of SonarQube
  │   ├── setup-keycloak.sh        # Scripts for local setup of keycloak
  │   └── setup-argocd.sh       # Scripts for local setup of ArgoCD
  ├── helm                      # Helm charts for deployment (if applicable)
  ├── config                    # Configuration files (e.g., vault configuration, ArgoCD setup)
  │   ├── sonar-config.json     # SonarQube configuration files
  │   └── keycloak-config.hcl      # keycloak config files
  └── README.md                 # Documentation for setting up and using the local dev environment

I would like to have microk8s running locally.
Then all components being deployed on it, like I need an gitea Source Control, them and ArgoCD for GitOps deploy, them a cluster to have my service deployed on and later other thinks like JMeter, Sonar, Harbor, Jaeger, Prometheus, etc..


This is a great plan for setting up a comprehensive local development environment. Here's how you can proceed to implement the structure and configuration:

---

### 1. **Set Up Directory Structure**

Create the directory structure as outlined in your request:

```bash
mkdir -p local-dev-environment/{k8s,terraform,scripts,helm,config}
touch local-dev-environment/{docker-compose.yml,README.md}
touch local-dev-environment/k8s/{sonarqube.yaml,argocd.yaml,keycloak.yaml}
touch local-dev-environment/terraform/{sonarqube.tf,argocd.tf,keycloak.tf}
touch local-dev-environment/scripts/{setup-sonar.sh,setup-argocd.sh,setup-keycloak.sh}
touch local-dev-environment/config/{sonar-config.json,keycloak-config.hcl}
```

---

### 2. **Step-by-Step Configuration**

#### **MicroK8s Setup**

Install and configure MicroK8s:

1. **Install MicroK8s:**
    
    ```bash
    sudo snap install microk8s --classic
    ```
    
2. **Enable Required Add-ons:**
    
    ```bash
    microk8s enable dns storage ingress metallb:192.168.15.100-192.168.15.200
    ```
    

#### **Source Control: Gitea**

Define the Kubernetes manifest for Gitea in `k8s/gitea.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitea
  namespace: local-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gitea
  template:
    metadata:
      labels:
        app: gitea
    spec:
      containers:
        - name: gitea
          image: gitea/gitea:latest
          ports:
            - containerPort: 3000
              name: http
---
apiVersion: v1
kind: Service
metadata:
  name: gitea
  namespace: local-dev
spec:
  selector:
    app: gitea
  ports:
    - port: 3000
      targetPort: 3000
      protocol: TCP
```

Deploy it:

```bash
kubectl apply -f k8s/gitea.yaml
```

---

#### **ArgoCD**

Set up the ArgoCD manifest in `k8s/argocd.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: argocd
  namespace: local-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: argocd
  template:
    metadata:
      labels:
        app: argocd
    spec:
      containers:
        - name: argocd-server
          image: argoproj/argocd:v2.7.0
          ports:
            - containerPort: 8080
              name: http
```

---

#### **Terraform**

Write Terraform configurations for automating the deployment of Gitea, ArgoCD, and other components in the `terraform/` directory. For example:

`terraform/gitea.tf`:

```hcl
provider "kubernetes" {
  config_path = "~/.kube/config"
}

resource "kubernetes_deployment" "gitea" {
  metadata {
    name = "gitea"
    namespace = "local-dev"
  }
  spec {
    replicas = 1
    selector {
      match_labels = {
        app = "gitea"
      }
    }
    template {
      metadata {
        labels = {
          app = "gitea"
        }
      }
      spec {
        container {
          name  = "gitea"
          image = "gitea/gitea:latest"
          port {
            container_port = 3000
          }
        }
      }
    }
  }
}
```

Initialize and apply Terraform:

```bash
cd terraform
terraform init
terraform apply
```

---

#### **Shell Scripts**

Automate deployments using `setup-*.sh` scripts. Example `scripts/setup-gitea.sh`:

```bash
#!/bin/bash
kubectl apply -f k8s/gitea.yaml
echo "Gitea deployed successfully!"
```

Make scripts executable:

```bash
chmod +x scripts/*.sh
```

---

#### **Helm**

Use Helm charts for tools that provide official charts. For example, add ArgoCD's Helm chart for deployment:

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd --namespace local-dev
```

---

#### **Configuration**

Place custom configuration files in `config/`. For example, `config/sonar-config.json` might contain:

```json
{
  "sonar.projectKey": "example",
  "sonar.sources": ".",
  "sonar.host.url": "http://localhost:9000",
  "sonar.login": "admin"
}
```

---

### 3. **README.md**

Document all steps in `README.md`:

```markdown
# Local Development Environment

## Prerequisites
- Docker
- MicroK8s
- Terraform
- Helm

## Setup Steps
1. Install MicroK8s and enable required add-ons.
2. Use `scripts/setup-*.sh` to deploy services.
3. Configure GitOps using ArgoCD.
4. Access services using the configured ingress.

## Directory Structure
Explain the purpose of each folder and file here.
```

---

### 4. **Future Additions**

- **Monitoring:** Add Prometheus and Grafana.
- **Registry:** Deploy Harbor for container image management.
- **Tracing:** Include Jaeger for distributed tracing.
- **Load Testing:** Deploy JMeter for performance testing.

Would you like me to help with more detailed configuration or any specific component?

Yes I would like to have more detailed configuration and in the end one script that start the whole setup process.

Here’s an enhanced version of the setup with detailed configurations and a single script to start the whole process.

---

### **Detailed Configuration**

#### **1. Kubernetes Namespace**

First, create a namespace for the local development environment:

`k8s/namespace.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: local-dev
```

Apply this file:

```bash
kubectl apply -f k8s/namespace.yaml
```

---

#### **2. Gitea**

Create a Kubernetes manifest for Gitea in `k8s/gitea.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitea
  namespace: local-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gitea
  template:
    metadata:
      labels:
        app: gitea
    spec:
      containers:
        - name: gitea
          image: gitea/gitea:latest
          ports:
            - containerPort: 3000
              name: http
---
apiVersion: v1
kind: Service
metadata:
  name: gitea
  namespace: local-dev
spec:
  selector:
    app: gitea
  ports:
    - port: 3000
      targetPort: 3000
      protocol: TCP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: gitea-ingress
  namespace: local-dev
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: gitea.local.dev
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: gitea
                port:
                  number: 3000
```

---

#### **3. ArgoCD**

Define `k8s/argocd.yaml` for deploying ArgoCD:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: argocd
  namespace: local-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: argocd
  template:
    metadata:
      labels:
        app: argocd
    spec:
      containers:
        - name: argocd-server
          image: argoproj/argocd:v2.7.0
          ports:
            - containerPort: 8080
              name: http
---
apiVersion: v1
kind: Service
metadata:
  name: argocd
  namespace: local-dev
spec:
  selector:
    app: argocd
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-ingress
  namespace: local-dev
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: argocd.local.dev
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: argocd
                port:
                  number: 8080
```

---

#### **4. SonarQube**

Add a Kubernetes manifest for SonarQube in `k8s/sonarqube.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sonarqube
  namespace: local-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sonarqube
  template:
    metadata:
      labels:
        app: sonarqube
    spec:
      containers:
        - name: sonarqube
          image: sonarqube:community
          ports:
            - containerPort: 9000
              name: http
---
apiVersion: v1
kind: Service
metadata:
  name: sonarqube
  namespace: local-dev
spec:
  selector:
    app: sonarqube
  ports:
    - port: 9000
      targetPort: 9000
      protocol: TCP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sonarqube-ingress
  namespace: local-dev
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: sonarqube.local.dev
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: sonarqube
                port:
                  number: 9000
```

---

#### **5. Scripts**

Create automation scripts in the `scripts` folder:

`scripts/setup-environment.sh`:

```bash
#!/bin/bash
set -e

# Define paths
BASE_DIR=$(dirname "$0")/..
K8S_DIR="$BASE_DIR/k8s"

echo "Applying Kubernetes namespace..."
kubectl apply -f "$K8S_DIR/namespace.yaml"

echo "Deploying Gitea..."
kubectl apply -f "$K8S_DIR/gitea.yaml"

echo "Deploying ArgoCD..."
kubectl apply -f "$K8S_DIR/argocd.yaml"

echo "Deploying SonarQube..."
kubectl apply -f "$K8S_DIR/sonarqube.yaml"

echo "Setup completed! Access services via:"
echo "- Gitea: http://gitea.local.dev"
echo "- ArgoCD: http://argocd.local.dev"
echo "- SonarQube: http://sonarqube.local.dev"
```

Make it executable:

```bash
chmod +x scripts/setup-environment.sh
```

---

### **Single Command to Start the Setup**

Run the following command from the root of your project:

```bash
scripts/setup-environment.sh
```

This will:

1. Create the `local-dev` namespace.
2. Deploy Gitea, ArgoCD, and SonarQube to MicroK8s.
3. Configure Ingress for accessing services.

---

### **Validation**

1. Ensure MicroK8s is running:
    
    ```bash
    microk8s status --wait-ready
    ```
    
2. Verify deployments:
    
    ```bash
    kubectl get all -n local-dev
    ```
    
3. Access services in your browser via `http://gitea.local.dev`, `http://argocd.local.dev`, and `http://sonarqube.local.dev`.

Would you like further refinements or additional components like Keycloak, Prometheus, or Jaeger added to the setup?

additional components like Keycloak, Prometheus, or Jaeger added to the setup.

Here’s an updated configuration that includes **Keycloak**, **Prometheus**, and **Jaeger** to the local development environment. Each component is integrated with Kubernetes manifests and automation scripts.

---

### **Updated Configuration**

#### **1. Keycloak**

`k8s/keycloak.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: keycloak
  namespace: local-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: keycloak
  template:
    metadata:
      labels:
        app: keycloak
    spec:
      containers:
        - name: keycloak
          image: quay.io/keycloak/keycloak:latest
          args: ["start-dev"]
          env:
            - name: KEYCLOAK_ADMIN
              value: admin
            - name: KEYCLOAK_ADMIN_PASSWORD
              value: admin
          ports:
            - containerPort: 8080
              name: http
---
apiVersion: v1
kind: Service
metadata:
  name: keycloak
  namespace: local-dev
spec:
  selector:
    app: keycloak
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: keycloak-ingress
  namespace: local-dev
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: keycloak.local.dev
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: keycloak
                port:
                  number: 8080
```

---

#### **2. Prometheus**

`k8s/prometheus.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: local-dev
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
      - job_name: 'kubernetes-pods'
        kubernetes_sd_configs:
          - role: pod
        relabel_configs:
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
            action: keep
            regex: true
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
            action: replace
            target_label: __metrics_path__
            regex: (.+)
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port]
            action: replace
            target_label: __address__
            regex: ([^:]+):(.+)
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: local-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
        - name: prometheus
          image: prom/prometheus:latest
          args:
            - "--config.file=/etc/prometheus/prometheus.yml"
          volumeMounts:
            - name: config-volume
              mountPath: /etc/prometheus
      volumes:
        - name: config-volume
          configMap:
            name: prometheus-config
---
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: local-dev
spec:
  selector:
    app: prometheus
  ports:
    - port: 9090
      targetPort: 9090
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prometheus-ingress
  namespace: local-dev
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: prometheus.local.dev
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: prometheus
                port:
                  number: 9090
```

---

#### **3. Jaeger**

`k8s/jaeger.yaml`:

```yaml
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: jaeger
  namespace: local-dev
spec:
  strategy: all-in-one
  ingress:
    enabled: true
    annotations:
      nginx.ingress.kubernetes.io/rewrite-target: /
    hosts:
      - jaeger.local.dev
  storage:
    type: memory
```

Install the Jaeger Operator first:

```bash
kubectl create namespace observability
kubectl apply -n observability -f https://github.com/jaegertracing/jaeger-operator/releases/latest/download/jaeger-operator.yaml
```

---

#### **4. Script Updates**

Update `scripts/setup-environment.sh` to include these new components:

```bash
#!/bin/bash
set -e

# Define paths
BASE_DIR=$(dirname "$0")/..
K8S_DIR="$BASE_DIR/k8s"

echo "Applying Kubernetes namespace..."
kubectl apply -f "$K8S_DIR/namespace.yaml"

echo "Deploying Gitea..."
kubectl apply -f "$K8S_DIR/gitea.yaml"

echo "Deploying ArgoCD..."
kubectl apply -f "$K8S_DIR/argocd.yaml"

echo "Deploying SonarQube..."
kubectl apply -f "$K8S_DIR/sonarqube.yaml"

echo "Deploying Keycloak..."
kubectl apply -f "$K8S_DIR/keycloak.yaml"

echo "Deploying Prometheus..."
kubectl apply -f "$K8S_DIR/prometheus.yaml"

echo "Deploying Jaeger Operator..."
kubectl apply -n observability -f https://github.com/jaegertracing/jaeger-operator/releases/latest/download/jaeger-operator.yaml

echo "Deploying Jaeger..."
kubectl apply -f "$K8S_DIR/jaeger.yaml"

echo "Setup completed! Access services via:"
echo "- Gitea: http://gitea.local.dev"
echo "- ArgoCD: http://argocd.local.dev"
echo "- SonarQube: http://sonarqube.local.dev"
echo "- Keycloak: http://keycloak.local.dev"
echo "- Prometheus: http://prometheus.local.dev"
echo "- Jaeger: http://jaeger.local.dev"
```

---

#### **5. Test the Setup**

Run the setup script:

```bash
scripts/setup-environment.sh
```

---

### **Next Steps**

If you want other components like **Harbor** (container registry) or **JMeter**, let me know, and I can extend this setup further.

Can you explain the microk8s part, any detail, like setup, about IP setup, and any additional config on my host machine

Here’s a detailed explanation of setting up **MicroK8s** with configurations for networking, IP setup, and necessary configurations for the host machine. This will ensure your environment is ready for deploying services like Gitea, ArgoCD, SonarQube, Keycloak, Prometheus, and Jaeger.

---

### **1. MicroK8s Installation**

#### Install MicroK8s

Run the following commands on your host machine (Ubuntu preferred):

```bash
sudo snap install microk8s --classic
```

#### Add Your User to the MicroK8s Group

```bash
sudo usermod -a -G microk8s $USER
sudo chown -R $USER ~/.kube
newgrp microk8s
```

#### Verify Installation

Check the status of MicroK8s:

```bash
microk8s status --wait-ready
```

---

### **2. Enable Add-ons**

MicroK8s includes several built-in add-ons for essential services like DNS, Ingress, and storage. Enable these add-ons:

```bash
microk8s enable dns ingress storage metallb:192.168.15.100-192.168.15.200
```

#### Key Add-ons Explained:

1. **DNS**: Resolves internal Kubernetes service names.
2. **Ingress**: Manages HTTP and HTTPS access to services.
3. **Storage**: Provides a default `hostPath` storage class for local development.
4. **MetalLB**: Assigns external IPs for LoadBalancer services. Here, a range of IPs (`192.168.15.100-192.168.15.200`) is specified for your local network.

---

### **3. Networking and IP Setup**

#### Reserve IP Range for MetalLB

Make sure the IP range (`192.168.15.100-192.168.15.200`) is not in use by other devices on your local network. This can be done via:

- Adjusting the DHCP range on your router.
- Reserving these IPs explicitly in your router’s settings.

#### Configure Host File for Local DNS

Add domain mappings to your `/etc/hosts` file:

```bash
sudo nano /etc/hosts
```

Add the following lines:

```
192.168.15.100 gitea.local.dev
192.168.15.101 argocd.local.dev
192.168.15.102 sonarqube.local.dev
192.168.15.103 keycloak.local.dev
192.168.15.104 prometheus.local.dev
192.168.15.105 jaeger.local.dev
```

Save and close.

---

### **4. Exposing MicroK8s Dashboard (Optional)**

You can enable the Kubernetes dashboard for monitoring:

```bash
microk8s enable dashboard
```

Access the dashboard:

```bash
microk8s kubectl proxy
```

Then open: [http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/)

---

### **5. Namespace Configuration**

To separate workloads, create a dedicated namespace:

```bash
microk8s kubectl create namespace local-dev
```

---

### **6. Persistent Storage**

MicroK8s uses `hostPath` by default. For production-like setups, create a PersistentVolume (PV) and PersistentVolumeClaim (PVC) for each service.

Example for Gitea: `k8s/gitea-storage.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: gitea-pv
  namespace: local-dev
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/gitea
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: gitea-pvc
  namespace: local-dev
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

Apply the storage:

```bash
microk8s kubectl apply -f k8s/gitea-storage.yaml
```

---

### **7. Ingress Configuration**

The `nginx` ingress add-on is enabled by default in MicroK8s. Use it to expose services over `http://<hostname>` or `http://<IP>`.

Example ingress for Gitea: `k8s/gitea-ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: gitea-ingress
  namespace: local-dev
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: gitea.local.dev
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: gitea
                port:
                  number: 3000
```

Apply the ingress:

```bash
microk8s kubectl apply -f k8s/gitea-ingress.yaml
```

---

### **8. Start All Services**

Run the setup script (`scripts/setup-environment.sh`) after configuring MicroK8s:

```bash
bash scripts/setup-environment.sh
```

---

### **9. Host Machine Requirements**

- **CPU/RAM**: Ensure at least 4 cores and 8 GB of RAM are available for running MicroK8s and services.
- **Firewall/Ports**: Open necessary ports for MetalLB IPs (e.g., 80/443 for HTTP/HTTPS, 9090 for Prometheus, etc.).
- **Docker**: For building and managing container images, install Docker if required:
    
    ```bash
    sudo apt update
    sudo apt install docker.io
    ```
    

---

### **10. Verification**

After deployment, verify each service:

1. Access the service URLs defined in your `/etc/hosts` file.
2. Use `kubectl get pods -n local-dev` to ensure all pods are running.

---

This setup ensures your environment is ready to deploy additional tools like **Harbor**, **JMeter**, etc. Let me know if you want their configurations added!

