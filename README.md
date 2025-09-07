<div align="center">

# 🎨 End-to-End DevOps Project: A Complete A-to-Z Guide ✨

**This repository contains a complete, step-by-step tutorial for building a fully automated CI/CD pipeline using Docker, Kubernetes, GitHub Actions, and Argo Rollouts.**

</div>

<div align="center">

[![CI Pipeline Status](https://github.com/sudlo/CI-CD/actions/workflows/ci-pipeline.yml/badge.svg)](https://github.com/sudlo/CI-CD/actions/workflows/ci-pipeline.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

</div>

Welcome! This guide is designed to be a single source of truth for creating a modern DevOps workflow. We will start with an empty cloud server and finish with a system where your code automatically deploys to a Kubernetes cluster moments after you push it to GitHub.

---

### ## 🛠️ The Tech Stack

<div align="center">
  <a href="https://www.python.org" target="_blank" rel="noreferrer">
    <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python">
  </a>
  <a href="https://flask.palletsprojects.com/" target="_blank" rel="noreferrer">
    <img src="https://img.shields.io/badge/Flask-000000?style=for-the-badge&logo=flask&logoColor=white" alt="Flask">
  </a>
  <a href="https://www.docker.com/" target="_blank" rel="noreferrer">
    <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker">
  </a>
  <a href="https://kubernetes.io" target="_blank" rel="noreferrer">
    <img src="https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white" alt="Kubernetes">
  </a>
  <a href="https://argo-cd.readthedocs.io/" target="_blank" rel="noreferrer">
    <img src="https://img.shields.io/badge/Argo%20CD-EF7B4D?style=for-the-badge&logo=argo&logoColor=white" alt="Argo CD">
  </a>
</div>

---

### ## 🚀 The Final Workflow

This is the automated pipeline we are going to build.

![DevOps Workflow Diagram](./assets/workflow-diagram.jpg)

---

## ## Phase 1: ☁️ Setting Up The Environment ✨

We'll start by preparing a cloud server on Azure with all the tools we need.

#### **1.1 Create the Azure VM**
1.  Log in to the **Azure Portal** and create a new **Virtual machine**.
2.  Use these settings:
    * **Image**: `Windows Server 2022 Datacenter: Azure Edition - Gen2`.
    * **Size**: A size that supports nested virtualization, like **`Standard_D4s_v3`**. This is critical.
    * **Administrator account**: Create a username and password.
    * **Inbound port rules**: Allow **`RDP (3389)`**.
3.  Create the VM and connect to it using Remote Desktop. 
    
#### **1.2 Configure the Server**
Inside your new VM, we need to prepare the operating system.
1.  Open **Server Manager** (it opens on startup) > **Local Server**.
2.  Turn **IE Enhanced Security Configuration** to **Off**.
3.  Open **PowerShell (Admin)** and install the **Chocolatey** package manager to make software installation easy:
    ```powershell
    Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('[https://community.chocolatey.org/install.ps1](https://community.chocolatey.org/install.ps1)'))
    ```
4.  Next, enable **Hyper-V** (the virtualization tool for Kubernetes) and restart the VM.
    ```powershell
    Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
    ```
5.  After the VM restarts, reconnect to it.

#### **1.3 Install All DevOps Tools**
1.  Open **PowerShell (Admin)** again.
2.  Install **Docker**, **Kubernetes tools**, **Git**, and **VS Code** using Chocolatey.
    ```powershell
    # Install Docker Engine and the command-line tool
    choco install docker-engine -y
    
    # Install Kubernetes command-line tool (kubectl)
    choco install kubernetes-cli -y
    
    # Install Minikube, which creates our local K8s cluster
    choco install minikube -y
    
    # Install Git and VS Code
    choco install git vscode -y
    ```
3.  Close and reopen PowerShell for all changes to take effect.

#### **1.4 Start Your Kubernetes Cluster**
1.  In PowerShell, tell Minikube to create your Kubernetes cluster. This can take several minutes.
    ```powershell
    minikube start --driver=hyperv
    ```
2.  Verify your cluster is running:
    ```powershell
    kubectl get nodes
    ```
    You should see one node with a status of **`Ready`**. Your environment is now fully prepared!

---

## ## Phase 2: 📝 Project & Code Setup ✨

Now let's create our application files and push them to this GitHub repository.

1.  **Clone This Repository**: In PowerShell, navigate to a projects folder (e.g., `C:\projects`) and clone this repo.
    ```powershell
    cd C:\projects
    git clone [https://github.com/sudlo/CI-CD.git](https://github.com/sudlo/CI-CD.git)
    cd CI-CD
    ```
2.  **Create the Application Files**: Open the folder in VS Code and create the following files.

    * **`app.py`**:
        ```python
        from flask import Flask
        app = Flask(__name__)
        @app.route('/')
        def hello():
            return "Hello from our Automated Pipeline! 🚀"
        if __name__ == '__main__':
            app.run(host='0.0.0.0', port=80)
        ```
    * **`requirements.txt`**:
        ```
        Flask==2.2.2
        ```
    * **`Dockerfile`**:
        ```dockerfile
        FROM python:3.9-slim
        WORKDIR /app
        COPY requirements.txt .
        RUN pip install --no-cache-dir -r requirements.txt
        COPY . .
        EXPOSE 80
        CMD ["python", "app.py"]
        ```
3.  **Push the Initial Files**: In the terminal, commit and push these new files.
    ```bash
    git add .
    git commit -m "Initial commit: Add application code and Dockerfile"
    git push
    ```

---

## ## Phase 3: 🤖 The CI Pipeline (GitHub Actions) ✨

Next, we'll set up the automation to build our Docker image.

#### **3.1 Get Docker Hub Credentials**
Our pipeline needs to log in to Docker Hub. For that, we need a Personal Access Token (PAT).
1.  Log in to **Docker Hub**.
2.  Click your **Username > Account Settings > Personal access tokens**.
3.  Click **"New Access Token"**.
4.  Give it a name (e.g., `github-actions-token`) and set permissions to **"Read, Write, Delete"**.
5.  Click **"Generate"** and **COPY the token immediately**. You will not see it again. 

#### **3.2 Set GitHub Secrets**
We need to securely store the Docker Hub credentials in GitHub.
1.  In this repository, go to **Settings > Secrets and variables > Actions**.
2.  Create two new repository secrets:
    * **`DOCKERHUB_USERNAME`**: Your Docker Hub username.
    * **`DOCKERHUB_TOKEN`**: The access token you just copied.

#### **3.3 Create the Workflow File**
Create the folder `.github/workflows/` and add the file `ci-pipeline.yml` with this content:
```yaml
name: CI Pipeline - Build and Push Docker Image
on:
  push:
    branches: [ "main" ]
jobs:
  build_and_push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Build and Push Docker Image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/ci-cd:${{ github.sha }}
```

#### **3.4 Push and Verify**
Commit and push the new workflow file. Go to the **"Actions"** tab in GitHub to watch it run. It should succeed with a green checkmark!

---

## ## Phase 4: 🛰️ The CD Pipeline (ArgoCD & Argo Rollouts) ✨

Now we set up automatic, advanced deployments to Kubernetes.

#### **4.1 Create Kubernetes Manifests**
Create the folder `k8s/` and add these two files:
* **`deployment.yaml`** (This will be an Argo Rollouts object):
    ```yaml
    apiVersion: argoproj.io/v1alpha1
    kind: Rollout
    metadata:
      name: ci-cd-app
    spec:
      replicas: 4
      selector:
        matchLabels:
          app: ci-cd
      template:
        metadata:
          labels:
            app: ci-cd
        spec:
          containers:
          - name: web
            image: sudlo/ci-cd:latest # Placeholder image
            ports:
            - containerPort: 80
      strategy:
        canary:
          steps:
          - setWeight: 25
          - pause: { duration: 1m }
          - setWeight: 75
          - pause: { duration: 1m }
    ```
* **`service.yaml`**:
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: ci-cd-service
    spec:
      selector:
        app: ci-cd
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
      type: NodePort
    ```
Commit and push these files.

#### **4.2 Install Argo CD & Argo Rollouts**
In your VM's PowerShell terminal, install both Argo controllers to your Kubernetes cluster:
```powershell
# Install Argo CD
kubectl create namespace argocd
kubectl apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)

# Install Argo Rollouts
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f [https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml](https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml)
```

#### **4.3 Access the Argo CD UI**
1.  **Open a new, separate terminal** and run this port-forward command. Keep it running.
    ```powershell
    kubectl port-forward svc/argocd-server -n argocd 8080:443
    ```
2.  In your original terminal, get the initial admin password:
    ```powershell
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
    ```
3.  Open a browser in your VM to `https://localhost:8080`. Log in with username `admin` and the password you just retrieved.

#### **4.4 Connect Your Application**
In the Argo CD UI, click **`+ NEW APP`** and fill in the details:
* **Application Name**: `ci-cd-app`
* **Project Name**: `default`
* **Sync Policy**: `Automatic`, and check the boxes for `Prune Resources` and `Self Heal`.
* **Repository URL**: `https://github.com/sudlo/CI-CD.git`
* **Revision**: `HEAD`
* **Path**: `k8s`
* **Cluster URL**: `https://kubernetes.default.svc`
* **Namespace**: `default`

Click **`CREATE`**. ArgoCD will now automatically deploy your application using the initial `Rollout` manifest.

---

## ## Phase 5: 🎉 Go Bonkers! Build & Destroy! ✨

It's time for the grand finale! Let's see that powerful pipeline in action, from a simple code change to a full-blown Canary deployment.

1.  **Make a Code Change**: In VS Code, edit your `app.py` file. Maybe add an emoji, change the greeting, or even break something on purpose to test the `SELF HEAL` feature!
2.  **Push the Code**: Save your changes, then in your terminal:
    ```bash
    git add .
    git commit -m "feat: Go bonkers with a new app version!"
    git push
    ```
3.  **Watch CI (GitHub Actions)**: Head over to the **"Actions"** tab on GitHub. Your CI pipeline will roar to life, building a fresh Docker image with a brand new, unique tag (the commit SHA). Make sure to **copy this new image tag** once the build is green.
4.  **Update the Manifest (GitOps Magic)**: In `k8s/deployment.yaml`, update the `image` line with the new tag you just copied:
    ```yaml
    image: sudlo/ci-cd:PASTE_THE_NEW_TAG_HERE
    ```
5.  **Push the Manifest Change**: This is the GitOps trigger!
    ```bash
    git add .
    git commit -m "chore: Initiating glorious canary deployment!"
    git push
    ```
6.  **Watch CD (ArgoCD & Argo Rollouts)**: Open the ArgoCD UI. You'll see your application go `OutOfSync` as it detects the change. Then, Argo Rollouts will take over, initiating your carefully crafted Canary deployment strategy. To witness every step of the traffic shift and pod creation in real-time, keep this terminal command running:
    ```bash
    kubectl argo rollouts get rollout ci-cd-app --watch
    ```
7.  **Verify & Celebrate**: Once the rollout is `Healthy` and `Synced`, run `minikube service ci-cd-service` in your terminal. A browser will pop open, displaying your latest, greatest (or perhaps intentionally broken for testing!) application version.

---

### ## 💥 You Did It! Go Bonkers! Build & Destroy! ✨