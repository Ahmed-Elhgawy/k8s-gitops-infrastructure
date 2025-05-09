# ☸️ GitOps Kubernetes Deployment with Argo CD & Sealed Secrets

This project demonstrates a full **GitOps-based Kubernetes deployment pipeline** with:

- 💻 Kubernetes cluster provisioned via **Ansible**
- 🔐 Secret management using **Sealed Secrets**
- 🚀 Continuous deployment using **Argo CD**
- 🗃 Persistent state with **MongoDB StatefulSet**
- 📦 Application deployment via K8s manifests

---

## 📚 Project Overview

### 🔧 Tools Used

- **Ansible** – to automate Kubernetes control-plane setup
- **Helm** – to install sealed-secrets and Argo CD
- **kubeseal** – to safely store secrets in Git
- **Argo CD** – to sync manifests from Git to cluster
- **Kubernetes** – to orchestrate the deployment
- **MongoDB** – deployed using StatefulSet and PVCs

---

## 📁 Project Structure

```bash
.
├── ansible
│   ├── cluster
│   │   ├── handlers
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yml
│   ├── CRT
│   │   ├── handlers
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yml
│   ├── kube
│   │   ├── handlers
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yml
│   ├── ansible.cfg
│   ├── inventory
│   └── site.yaml
├── manifest
│   ├── application.yml
│   ├── mongodb.yaml
│   ├── sealed-secret.yaml
│   └── secret.yaml
└── README.md
```

---

## 🚀 Setup Workflow

### 1. ⚙️ Provision Control Plane (via Ansible)

```bash
cd ansible
ansible-playbook site.yaml
```

---

### 2. 📦 Install Argo CD and Sealed Secrets (via Helm)

```bash
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install sealed-secrets-controller sealed-secrets/sealed-secrets -n kube-system
helm install argo-cd bitnami/argo-cd -n argocd --create-namespace
```

---

### 3. 🔐 Seal Secrets

```bash
kubectl get secret <sealed-secrets-keyxxxxx> -n kube-system -o jsonpath="{.data['tls\.crt']}" | base64 -d > pub-cert.pem
kubeseal --cert pub-cert.pem -o yaml < secret.yaml > sealed-secret.yaml
```

---

### 4. 🧩 Push Manifests to GitHub

Commit all Kubernetes manifests and sealed secrets (without secret manifest file) to your GitHub repo.

---

### 5. 🔄 Apply Argo CD App

Once Argo cd deployed, you can open the Argo CD UI to create, observe and manage your app visually:

```bash
kubectl port-forward svc/argo-cd-argocd-server -n argocd 8080:80
# or
kubectl edit svc/argo-cd-argocd-server
```
and change its type ot `NodePort` instead of `ClusterIP`

To login to Argo CD UI use username __admin__ and password the output of the following command
```bash
kubectl -n argocd get secret argocd-secret -o jsonpath="{.data.clearPassword}" | base64 -d
```
then add new application in Argo CD and add the repo URL that contain menifests file that will use to deploy the application 

---

## 🗂 MongoDB Resources

`mongodb.yaml` Deploys:
1. __mongo__ Service that used to connect to MongoDB pods
2. __mongodb__ Statefulsets
3. __my-storage-class__ Storage Class
4. __data-pv__ Presistent Volume

## 🛠 Application Resources

`application.yaml` Deploys:
1. __todo-service__ Service that used to application accessablefor users
2. __todo-deployment__ Application Deployment

---

🙌 Author

Ahmed Elhgawy – [GitHub](https://github.com/Ahmed-Elhgawy) | [LinkedIn](https://linkedin.com/in/ahmed-mahmoud-a16310268)
