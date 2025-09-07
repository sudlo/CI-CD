<div align="center">

# 🎨 End-to-End DevOps Project: A GitOps CI/CD Pipeline

**A fully automated CI/CD pipeline for a Python application using Docker, Kubernetes, GitHub Actions, and ArgoCD.**

</div>

<div align="center">

[![CI Pipeline Status](https://github.com/sudlo/CI-CD/actions/workflows/ci-pipeline.yml/badge.svg)](https://github.com/sudlo/CI-CD/actions/workflows/ci-pipeline.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

</div>

### ## 🛠️ Tech Stack

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

### ## 🚀 Workflow Overview

This diagram illustrates the complete CI/CD pipeline from code commit to deployment:

![DevOps Workflow Diagram](./assets/workflow-diagram.jpg)

1.  **CI (Continuous Integration)**: A push to the `main` branch triggers a **GitHub Actions** workflow. It builds the application into a **Docker** image and pushes it to Docker Hub.
2.  **CD (Continuous Deployment)**: **ArgoCD** monitors the Git repository. When it detects a change in the Kubernetes manifests, it automatically syncs the changes to the **Kubernetes** cluster.

---

### ## 📂 Project Structure

```bash
.
├── .github/
│   └── workflows/
│       └── ci-pipeline.yml
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── assets/
│   └── workflow-diagram.jpg
├── app.py
├── Dockerfile
└── requirements.txt
```

---

### ## 🔧 How to Replicate This Project

To run this project, you'll need a Kubernetes cluster and the following tools installed: `git`, `docker`, `kubectl`.

1.  **Fork & Clone the Repository**.
2.  **Set Up GitHub Secrets**: `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`.
3.  **Deploy ArgoCD** to your Kubernetes cluster.
4.  **Connect ArgoCD** to your forked repository.

Now, any change you make to the app code or manifests will automatically be built and deployed!