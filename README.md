# Kubernetes Basics Lab

> A hands-on beginner-friendly laboratory for learning Kubernetes fundamentals with kind, kubectl, and NGINX.

---

## Overview

This repository provides a small, practical Kubernetes laboratory designed to introduce the core concepts needed to deploy and manage containerized applications.

The lab uses [kind](https://kind.sigs.k8s.io/) to run a local Kubernetes cluster and deploys a simple NGINX application using standard Kubernetes manifests.

The focus is on understanding Kubernetes fundamentals rather than building a production-ready environment.

---

## Objectives

By completing this lab, you will learn how to:

- Create and manage a local Kubernetes cluster.
- Use `kubectl` to interact with Kubernetes.
- Create and use namespaces.
- Deploy applications using Deployments.
- Understand Pods and ReplicaSets.
- Expose applications using Services.
- Work with labels and selectors.
- Scale a Deployment.
- Perform a basic rolling update.
- Inspect and troubleshoot Kubernetes resources.

---

## Architecture

The lab uses a simple NGINX deployment running inside a dedicated Kubernetes namespace.

```text
                    Kubernetes Cluster
                           │
                           ▼
                    ┌─────────────┐
                    │  Namespace  │
                    │     web     │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │   Service   │
                    └──────┬──────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │    Deployment   │
                  └────────┬────────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
           ┌─────┐      ┌─────┐      ┌─────┐
           │ Pod │      │ Pod │      │ Pod │
           │NGINX│      │NGINX│      │NGINX│
           └─────┘      └─────┘      └─────┘
```

---

## Technologies

- Kubernetes
- [kind](https://kind.sigs.k8s.io/)
- [kubectl](https://kubernetes.io/docs/reference/kubectl/)
- NGINX
- YAML
- GitHub Actions

---

## Prerequisites

Before starting, make sure the following tools are installed:

- Docker
- kubectl
- kind
- Git

Verify the installations:

```bash
docker --version
kubectl version --client
kind version
git --version
```

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/delgadoroberto/kubernetes-basics-lab.git
cd kubernetes-basics-lab
```

### 2. Create the Kubernetes cluster

Create a local cluster using kind:

```bash
kind create cluster --name kubernetes-basics
```

Verify that the cluster is available:

```bash
kubectl cluster-info
```

Check the nodes:

```bash
kubectl get nodes
```

---

## Deploy the Lab

Apply the Kubernetes manifests:

```bash
kubectl apply -f manifests/
```

Verify the namespace:

```bash
kubectl get namespaces
```

Check the deployed resources:

```bash
kubectl get all -n web
```

---

## Explore the Deployment

List the Pods:

```bash
kubectl get pods -n web
```

List the Deployment:

```bash
kubectl get deployments -n web
```

List the ReplicaSets:

```bash
kubectl get replicasets -n web
```

List the Service:

```bash
kubectl get services -n web
```

---

## Scale the Application

Scale the NGINX Deployment to three replicas:

```bash
kubectl scale deployment nginx --replicas=3 -n web
```

Verify the Pods:

```bash
kubectl get pods -n web
```

---

## Perform a Rolling Update

Update the NGINX image:

```bash
kubectl set image deployment/nginx nginx=nginx:<version> -n web
```

Check the rollout status:

```bash
kubectl rollout status deployment/nginx -n web
```

Review the Deployment:

```bash
kubectl get deployment nginx -n web
```

---

## Troubleshooting

Inspect a Pod:

```bash
kubectl describe pod <pod-name> -n web
```

View container logs:

```bash
kubectl logs <pod-name> -n web
```

Inspect the Deployment:

```bash
kubectl describe deployment nginx -n web
```

Check all resources:

```bash
kubectl get all -n web
```

---

## Cleanup

Delete the Kubernetes resources:

```bash
kubectl delete -f manifests/
```

Delete the local kind cluster:

```bash
kind delete cluster --name kubernetes-basics
```

---

## Repository Structure

```text
kubernetes-basics-lab/
├── .github/
│   └── workflows/
│       └── yaml-lint.yml
├── docs/
│   └── kubernetes-basics.md
├── manifests/
│   ├── namespace.yaml
│   ├── deployment.yaml
│   └── service.yaml
├── .gitignore
├── LICENSE
└── README.md
```

---

## What This Lab Does Not Cover

This repository intentionally focuses on Kubernetes fundamentals.

The following topics are outside the scope of this lab:

- Helm
- Ingress
- Persistent Volumes
- StatefulSets
- RBAC
- Network Policies
- Kubernetes Operators
- Service Mesh
- Monitoring and Observability
- Production cluster administration

These topics can be explored in more advanced Kubernetes projects.

---

## Security Considerations

Although this is a beginner-level laboratory, some basic security principles should be considered when working with Kubernetes:

- Use minimal container images when possible.
- Avoid running containers as root.
- Do not store secrets directly in Git repositories.
- Keep container images updated.
- Avoid exposing Kubernetes services unnecessarily.
- Apply least-privilege principles when introducing RBAC.

Security hardening is intentionally outside the scope of this lab.

---

## Author

**Roberto Delgado**

*Cybersecurity Engineer*

Cybersecurity professional focused on cloud and infrastructure security, DevSecOps, vulnerability management, and security automation.

This repository is part of my technical portfolio, featuring hands-on projects that demonstrate secure engineering practices across cloud environments, Infrastructure as Code, container security, CI/CD, and security automation.

> **Practical cybersecurity. Secure automation. Continuous learning.**

---

## 📄 License

This project is licensed under the MIT License. See the `LICENSE` file for details.
