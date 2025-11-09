# Gitea on Kubernetes – Dev to Prod Transition

This repository demonstrates the deployment of **Gitea**, a lightweight self-hosted Git service, on **Kubernetes** using **Helm**, with a transition from a development setup (SQLite) to a production setup using **MySQL** and **Ingress** for external access.

---

## Objective

The goal of this lab is to:
- Deploy **Gitea** on Kubernetes using **Helm**.
- Replace the default SQLite database with an **external MySQL** database.
- Configure **persistent data storage** for Gitea.
- Expose Gitea publicly through an **Ingress** controller.
- Ensure the deployment is production-ready and reproducible.

---


## Step-by-Step Deployment Guide

### **1️. Deploy MySQL Database**

Apply the MySQL manifest to create a database service and pod:

```
kubectl apply -f mysql-deploy.yaml
```

Wait until the pod is ready:
```
kubectl get pods
```
### **2️. Configure and Deploy Gitea via Helm**

Ensure the Gitea Helm chart is available locally, then run:
```
helm upgrade --install gitea-test gitea-charts/gitea -f gitea/values.yaml
```

Once deployed, confirm that all pods are running:
```
kubectl get pods
```
### Expected output:
<img width="613" height="69" alt="image" src="https://github.com/user-attachments/assets/48d668cf-6805-4188-9f15-39edec7cabbc" />

### **3️. Access Gitea (Local Dev Test)**

Forward the Gitea service temporarily to localhost:
```
kubectl port-forward deployment/gitea-test 3000:3000
```
Output example:
```
Forwarding from 127.0.0.1:3000 -> 3000
Handling connection for 3000
```
Once port-forwarding is enabled, open **http://127.0.0.1:3000**  
You should see the Gitea welcome page like below:
<img width="620" height="655" alt="image" src="https://github.com/user-attachments/assets/caa8f771-3e5e-494c-a351-d8672caf57c3" />

### **4️. Configure Public Access via Ingress**

Apply the Ingress configuration:
```
kubectl apply -f gitea/ingress.yml
```

Add this entry to your Windows hosts file (C:\Windows\System32\drivers\etc\hosts):
```
127.0.0.1 gitea.local
```


After applying the Ingress file and updating the hosts file,  
you can access Gitea at **http://gitea.local**

### **Configuration Details:**

Helm Values (gitea/values.yaml)
```
gitea:
  admin:
    username: gitea_admin
    password: <your-secret-password>
    email: "gitea@local.domain"

  config:
    server:
      SSH_LISTEN_PORT: 2222
      ROOT_URL: http://gitea.local
    database:
      DB_TYPE: mysql
      HOST: mysql-service:3306
      NAME: gitea
      USER: gitea
      PASSWD: gitea123
```
- Gitea now connects to MySQL instead of SQLite.

- The ROOT_URL is updated to gitea.local for external exposure.

- Persistent storage is managed automatically by Helm.
---
### **Verification**

To verify everything works:
Go to **http://gitea.local**

Log in using:
```
Username: gitea_admin
Password: <your-secret-password>
```
Successfully logged in using the admin credentials.
<img width="621" height="627" alt="image" src="https://github.com/user-attachments/assets/ac57df0b-1435-45c7-a3cc-32a0f0865964" />


- Create a repository or organization to confirm data persistence.
  <img width="913" height="976" alt="image" src="https://github.com/user-attachments/assets/d550543e-993d-4250-ae74-9858a53270c1" />

- Restart pods and ensure data remains intact.
---

### *Cleanup*

To remove all resources:
```
helm uninstall gitea-test
kubectl delete -f mysql-deploy.yaml
kubectl delete -f gitea/ingress.yml
```
