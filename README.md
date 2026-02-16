# 🚀 ArgoCD GitOps Local Lab (Minikube + GitHub)

This guide contains the exact steps to set up a GitOps pipeline on your local machine using **Minikube**, **ArgoCD**, and this repository.

---

## 🛠️ Phase 1: Environment Setup

Start your local Kubernetes cluster and install the ArgoCD controller.

### 1. Start Minikube
```bash
minikube start
```
2. Install ArgoCD
Bash
# Create the namespace
```kubectl create namespace argocd```

# Apply the official manifests
```kubectl apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)```

3. Expose the API Server
Open a new terminal tab and run this to access the UI:

Bash
```kubectl port-forward svc/argocd-server -n argocd 8080:443```

🔐 Phase 2: Authentication
1. Get Admin Credentials
The default username is admin. To get the auto-generated password, run:

Bash
```kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo```

2. Login to UI
Go to https://localhost:8080 and use the credentials above.

📦 Phase 3: The Application Manifest
Create a file named app-deployment.yaml in this repo with the following content:
```
YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```
🔄 Phase 4: Connect Repo to ArgoCD
Apply this "Application" manifest to tell ArgoCD to watch this GitHub repo. Replace YOUR_GITHUB_USER with your actual username.
YAML
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-gitops-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: [https://github.com/djkaz/argocd-local-test.git](https://github.com/djkaz/argocd-local-test.git)
    targetRevision: HEAD
    path: .
  destination:
    server: [https://kubernetes.default.svc](https://kubernetes.default.svc)
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```
🧪 Phase 5: Testing the "Magic"
Check Status: Run kubectl get pods. You should see 2 Nginx pods.

Make a Change: Edit app-deployment.yaml in GitHub and change replicas: 2 to replicas: 4.

Commit & Push: Push the change to the main branch.

Watch: In less than 3 minutes (or immediately if you click 'Refresh' in Argo UI), ArgoCD will spin up 2 additional pods automatically.

🧹 Cleanup
To delete everything and start over:

Bash
```minikube delete```

Lab maintained by djkaz
