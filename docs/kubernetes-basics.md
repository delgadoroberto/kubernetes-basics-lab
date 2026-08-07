# Kubernetes Basics

This document introduces the core Kubernetes concepts used throughout the lab.

The goal is to understand how Kubernetes organizes, deploys, exposes, and manages containerized applications.

---

## Table of Contents

- [What Is Kubernetes?](#what-is-kubernetes)
- [Kubernetes Architecture](#kubernetes-architecture)
- [Cluster](#cluster)
- [Namespace](#namespace)
- [Pod](#pod)
- [Deployment](#deployment)
- [ReplicaSet](#replicaset)
- [Labels and Selectors](#labels-and-selectors)
- [Service](#service)
- [How the Resources Work Together](#how-the-resources-work-together)
- [kubectl](#kubectl)
- [Scaling](#scaling)
- [Rolling Updates](#rolling-updates)
- [Basic Troubleshooting](#basic-troubleshooting)
- [Key Takeaways](#key-takeaways)

---

## What Is Kubernetes?

Kubernetes is an open-source container orchestration platform designed to automate the deployment, scaling, and management of containerized applications.

Instead of manually starting and managing containers, Kubernetes allows you to describe the desired state of an application and continuously works to maintain that state.

For example, you can define that an application should run three replicas:

```yaml
spec:
  replicas: 3
```

Kubernetes then creates and manages the required Pods to maintain that desired state.

---

## Kubernetes Architecture

A Kubernetes environment consists of a cluster containing a control plane and one or more worker nodes.

At a high level:

```text
                        Kubernetes Cluster
                               │
                 ┌─────────────┴─────────────┐
                 │                           │
          Control Plane                 Worker Node
                 │                           │
       ┌─────────┼─────────┐          ┌──────┴──────┐
       │         │         │          │             │
   API Server  Scheduler  Controller  Pod           Pod
                           Manager
```

### Control Plane

The control plane manages the Kubernetes cluster and is responsible for making decisions about the desired state of the environment.

Important components include:

- API Server
- Scheduler
- Controller Manager
- etcd

For this beginner lab, the most important component to understand is the Kubernetes API Server.

### API Server

The API Server is the main entry point for interacting with Kubernetes.

Tools such as `kubectl` communicate with the API Server to create, inspect, update, and delete resources.

For example:

```bash
kubectl get pods
```

The command sends a request to the Kubernetes API Server, which returns information about the Pods in the current context.

### Worker Nodes

Worker nodes run application workloads.

They contain the components required to run containers, including:

- Kubelet
- Container runtime
- Network components

Pods are scheduled onto worker nodes by the Kubernetes control plane.

---

## Cluster

A Kubernetes cluster is a collection of machines that run Kubernetes workloads.

A cluster contains:

- A control plane
- One or more worker nodes

In this lab, [kind](https://kind.sigs.k8s.io/) is used to create a local Kubernetes cluster.

Create the cluster with:

```bash
kind create cluster --name kubernetes-basics
```

Check the available nodes:

```bash
kubectl get nodes
```

Example output:

```text
NAME                            STATUS   ROLES           AGE
kubernetes-basics-control-plane Ready    control-plane   1m
```

The exact output may vary depending on the Kubernetes and kind versions being used.

---

## Namespace

A Namespace provides logical isolation within a Kubernetes cluster.

Namespaces are useful for organizing and separating resources.

This lab uses a namespace called `web`.

The namespace is defined in:

```text
manifests/namespace.yaml
```

A basic Namespace manifest looks like:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: web
```

After applying the manifest, resources can be queried within the namespace:

```bash
kubectl get all -n web
```

The `-n web` option specifies that the command should operate against the `web` namespace.

---

## Pod

A Pod is the smallest deployable unit in Kubernetes.

A Pod represents one or more containers that are deployed together and share networking and storage resources.

In this lab, each Pod runs an NGINX container.

A simplified Pod relationship looks like:

```text
Pod
└── Container
    └── NGINX
```

Pods are generally not created directly for application management.

Instead, higher-level Kubernetes resources such as Deployments are used to manage them.

You can list Pods with:

```bash
kubectl get pods -n web
```

Inspect a specific Pod with:

```bash
kubectl describe pod <pod-name> -n web
```

View its logs with:

```bash
kubectl logs <pod-name> -n web
```

---

## Deployment

A Deployment provides declarative management for Pods.

It defines the desired state of an application and allows Kubernetes to maintain that state.

For example:

```yaml
spec:
  replicas: 2
```

This tells Kubernetes that two Pod replicas should be running.

The relationship can be represented as:

```text
Deployment
     │
     ▼
ReplicaSet
     │
     ├── Pod
     └── Pod
```

The Deployment is responsible for managing the ReplicaSet, while the ReplicaSet ensures that the requested number of Pods exists.

List Deployments with:

```bash
kubectl get deployments -n web
```

Inspect a Deployment with:

```bash
kubectl describe deployment nginx -n web
```

---

## ReplicaSet

A ReplicaSet maintains a stable number of identical Pods.

For example, if a Deployment specifies:

```yaml
replicas: 3
```

the associated ReplicaSet attempts to maintain three Pods.

If one Pod fails, Kubernetes can create another Pod to return the application to the desired state.

The relationship is:

```text
Deployment
     │
     ▼
ReplicaSet
     │
     ├── Pod 1
     ├── Pod 2
     └── Pod 3
```

ReplicaSets are normally managed indirectly through Deployments rather than created manually.

List ReplicaSets with:

```bash
kubectl get replicasets -n web
```

---

## Labels and Selectors

Labels are key-value pairs attached to Kubernetes resources.

They are used to identify and organize resources.

For example:

```yaml
metadata:
  labels:
    app: nginx
```

A Kubernetes Service can use a selector to identify Pods with a specific label:

```yaml
selector:
  app: nginx
```

This creates a relationship between the Service and the Pods.

Conceptually:

```text
Pod                     Service
┌─────────────┐         ┌─────────────┐
│ app: nginx  │ ◄────── │ app: nginx  │
└─────────────┘         └─────────────┘

┌─────────────┐
│ app: nginx  │ ◄────── Service selector
└─────────────┘
```

Labels and selectors are fundamental to how Kubernetes resources discover and interact with each other.

---

## Service

Pods are ephemeral. Their IP addresses can change when Pods are recreated.

A Service provides a stable network endpoint for accessing a group of Pods.

For example:

```text
                    Service
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
          Pod NGINX           Pod NGINX
```

The Service identifies the appropriate Pods using a selector:

```yaml
selector:
  app: nginx
```

The Service manifest is stored in:

```text
manifests/service.yaml
```

List Services with:

```bash
kubectl get services -n web
```

Inspect a Service with:

```bash
kubectl describe service nginx -n web
```

---

## How the Resources Work Together

The main resources in this lab work together as follows:

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
                         │ Deployment  │
                         └──────┬──────┘
                                │
                                ▼
                         ┌─────────────┐
                         │ ReplicaSet  │
                         └──────┬──────┘
                                │
                    ┌───────────┼───────────┐
                    ▼           ▼           ▼
                  Pod         Pod         Pod
                NGINX       NGINX       NGINX
                    ▲           ▲           ▲
                    │           │           │
                    └───────────┼───────────┘
                                │
                           ┌────┴────┐
                           │ Service │
                           └─────────┘
```

The flow is:

1. The Namespace provides logical organization.
2. The Deployment defines the desired application state.
3. The ReplicaSet maintains the requested number of Pods.
4. The Pods run the NGINX containers.
5. Labels identify the Pods.
6. The Service selects the Pods using those labels and provides a stable network endpoint.

---

## kubectl

`kubectl` is the command-line interface used to interact with Kubernetes clusters.

The general command structure is:

```bash
kubectl <command> <resource>
```

Common commands include:

### Get resources

```bash
kubectl get pods -n web
kubectl get deployments -n web
kubectl get replicasets -n web
kubectl get services -n web
```

### Describe resources

```bash
kubectl describe pod <pod-name> -n web
kubectl describe deployment nginx -n web
kubectl describe service nginx -n web
```

The `describe` command provides detailed information that is useful for troubleshooting.

### View logs

```bash
kubectl logs <pod-name> -n web
```

### Apply manifests

```bash
kubectl apply -f manifests/
```

The `apply` command creates or updates resources according to the configuration defined in the manifests.

### Delete resources

```bash
kubectl delete -f manifests/
```

---

## Scaling

One of the advantages of Deployments is the ability to easily scale applications.

For example, scale NGINX to three replicas:

```bash
kubectl scale deployment nginx --replicas=3 -n web
```

Verify the result:

```bash
kubectl get pods -n web
```

The expected state is three running Pods:

```text
NAME                     READY   STATUS    RESTARTS   AGE
nginx-xxxxxxxxxx-xxxxx   1/1     Running   0          1m
nginx-xxxxxxxxxx-xxxxx   1/1     Running   0          1m
nginx-xxxxxxxxxx-xxxxx   1/1     Running   0          1m
```

The exact Pod names will vary.

To scale the application back down:

```bash
kubectl scale deployment nginx --replicas=1 -n web
```

---

## Rolling Updates

Deployments support rolling updates when the application image or configuration changes.

For example:

```bash
kubectl set image deployment/nginx nginx=nginx:<version> -n web
```

Kubernetes gradually replaces the existing Pods with new Pods instead of stopping the entire application at once.

Check the rollout status:

```bash
kubectl rollout status deployment/nginx -n web
```

You can also inspect the Deployment:

```bash
kubectl get deployment nginx -n web
```

If necessary, a previous revision can be restored:

```bash
kubectl rollout undo deployment/nginx -n web
```

Check the rollout history with:

```bash
kubectl rollout history deployment/nginx -n web
```

---

## Basic Troubleshooting

When a Kubernetes workload does not behave as expected, start by checking the state of the resources.

### 1. Check Pods

```bash
kubectl get pods -n web
```

Look for statuses such as:

- `Running`
- `Pending`
- `CrashLoopBackOff`
- `ImagePullBackOff`
- `Error`

### 2. Describe the Pod

```bash
kubectl describe pod <pod-name> -n web
```

The Events section at the bottom of the output is particularly useful for identifying problems.

### 3. Check Logs

```bash
kubectl logs <pod-name> -n web
```

Logs can help identify application-level problems.

### 4. Check the Deployment

```bash
kubectl describe deployment nginx -n web
```

This can help identify replica, scheduling, or rollout problems.

### 5. Check the Service

```bash
kubectl describe service nginx -n web
```

Verify that the Service selector matches the labels assigned to the Pods.

For example:

```yaml
selector:
  app: nginx
```

should match:

```yaml
labels:
  app: nginx
```

A mismatch between selectors and labels can prevent a Service from routing traffic to the expected Pods.

---

## Key Takeaways

The most important concepts introduced in this lab are:

### Cluster

The environment where Kubernetes resources and workloads run.

### Namespace

A logical boundary used to organize and isolate resources within a cluster.

### Pod

The smallest deployable unit in Kubernetes and the execution environment for containers.

### Deployment

A declarative resource used to manage application Pods and updates.

### ReplicaSet

Maintains the desired number of Pod replicas.

### Labels

Key-value metadata used to identify and organize resources.

### Selectors

Mechanisms used by Kubernetes resources to identify other resources based on labels.

### Service

Provides a stable network endpoint for accessing a group of Pods.

### kubectl

The primary command-line tool used to interact with Kubernetes.

Understanding how these resources relate to each other provides the foundation for working with more advanced Kubernetes concepts.
