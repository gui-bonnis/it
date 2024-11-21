How can I manage local microk8s with k9

###### ChatGPT said:

ChatGPT

Managing your local MicroK8s cluster with K9s is a great way to monitor and manage Kubernetes resources using a terminal-based UI. Here's a guide to getting started with K9s and MicroK8s:

### Steps to Manage MicroK8s with K9s:

1. **Install K9s** K9s is a simple CLI tool that helps you interact with your Kubernetes clusters. To install K9s:
    
    - On Ubuntu/Debian:
        
        bash
        
        Copy code
        
        `curl -sS https://webinstall.dev/k9s | bash`
        
        Alternatively, you can use the package manager:
        
        bash
        
        Copy code
        
        `sudo snap install k9s`
        
    - On macOS (via Homebrew):
        
        bash
        
        Copy code
        
        `brew install k9s`
        
2. **Enable MicroK8s Add-ons** Ensure that your MicroK8s cluster is running and has necessary add-ons like DNS and storage:
    
    bash
    
    Copy code
    
    `microk8s enable dns storage`
    
3. **Check MicroK8s Status** Verify your MicroK8s setup:
    
    bash
    
    Copy code
    
    `microk8s status --wait-ready`
    
4. **Set Up `kubectl` Alias for MicroK8s** MicroK8s uses a built-in `kubectl`. To simplify working with Kubernetes commands and to ensure that K9s recognizes your MicroK8s cluster, you may want to alias `kubectl`:
    
    bash
    
    Copy code
    
    `alias kubectl='microk8s kubectl'`
    
5. **Configure kubeconfig for K9s** MicroK8s does not use the default kubeconfig file (`~/.kube/config`). To make K9s recognize your MicroK8s cluster, you can generate a `kubeconfig` file:
    
    bash
    
    Copy code
    
    `microk8s config > ~/.kube/config`
    
    Alternatively, run the following command to make K9s use MicroK8s directly without modifying your kubeconfig:
    
    bash
    
    Copy code
    
    `KUBECONFIG=$(microk8s config) k9s`
    
6. **Start K9s** Now, you can start K9s to monitor and manage your local MicroK8s cluster:
    
    bash
    
    Copy code
    
    `k9s`
    
    Once inside K9s, you'll see a terminal-based dashboard where you can navigate through namespaces, pods, services, deployments, and more using keyboard shortcuts.
    
7. **Useful K9s Shortcuts**
    
    - **`:q`**: Quit K9s.
    - **`:`**: Open the command bar.
    - **`d`**: Describe a resource.
    - **`l`**: Logs of a pod.
    - **`Enter`**: View details of a resource.

### Troubleshooting

- **Permission Issues**: If you face permission issues, ensure that your user is part of the `microk8s` group:
    
    bash
    
    Copy code
    
    `sudo usermod -a -G microk8s $USER sudo chown -R $USER ~/.kube newgrp microk8s`
    

With K9s, you can efficiently manage your MicroK8s cluster without needing to type long `kubectl` commands all the time.

4o