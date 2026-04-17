# POV-K8S

A collection of Kubernetes Proof-of-Concept (PoC) manifests and lab environments for testing microservices, multi-namespace architectures, and local cluster networking.

## 📂 Project Structure

*   **`kindsetup/` & `kindconfig/`**: Scripts and configurations to provision local Kubernetes clusters using **Kind**. Includes multi-region simulations (audc, jpdc, sgdc) and **MetalLB** setup for LoadBalancer support.
*   **`bookinfo-different-namespace/`**: A specialized lab demonstrating cross-namespace communication and FQDN configuration for the Istio Bookinfo app.
*   **`hashicups_works/`**: Deployment manifests for the Hashicups demo application (Frontend, Public API, Product API, and DB).
*   **`apps/`**: General-purpose manifests for various tools like Kubernetes Dashboard and sample counting apps.

---

## 🚀 Quick Start

### 1. Provision the Cluster
Choose a regional configuration and run the setup script:
```bash
chmod +x kindsetup/*.sh
./kindsetup/setupkindcluster134-audc-cluster.sh
```

### 2. Deploy an Application
To deploy the standard Bookinfo suite:
```bash
kubectl apply -f apps/bookinfo.yaml
```

To deploy the **Multi-Namespace Lab**:
```bash
cd bookinfo-different-namespace
# Follow the README inside this folder for specific instructions
```

### 3. Clean Up
To delete the cluster and resources:
```bash
./kindsetup/teardown.sh
```

---

## 🛠 Prerequisites
*   [Docker](https://www.docker.com/)
*   [Kind](https://kind.sigs.k8s.io/)
*   [kubectl](https://kubernetes.io/docs/tasks/tools/)