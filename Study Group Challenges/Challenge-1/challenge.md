# Deployment Configuration and Log Inspection in Kubernetes

Imagine you manage a Kubernetes cluster hosting applications for both development and production environments. Nodes in the cluster are labeled with either `environment=development` or `environment=production` to maintain resource isolation.

## Task 1: Deployment Configuration

Your task is to create deployment configurations for two applications using the `nginx:latest` image:

1. **Development Application:**
   - Deployment Name: `deployment-app`
   - Replicas: 1
   - Labels: `app=dev-app`
   - Affinity: Schedule on nodes labeled with `environment=development`.

2. **Production Application:**
   - Deployment Name: `production-app`
   - Replicas: 3
   - Labels: `app=prod-app`
   - Affinity: Schedule on nodes labeled with `environment=production`.

Write the necessary Kubernetes manifests to achieve this configuration. Provide a step-by-step solution.

## Task 2: Log Inspection

After successfully deploying the applications, you need to inspect the logs of the three pods within the production environment.

### Task

What command can you use to accomplish this?
