# Kubernetes Cluster Administration Task

As a Kubernetes cluster administrator, your responsibility is to generate a kubeconfig file for a developer. This file will grant them access to the cluster, enabling them to monitor deployments, inspect pods, and access pod logs.

## 1: Create a ServiceAccount

Create a ServiceAccount named `developer` in the `default` namespace.

## 2: Generate an API Token

Create an API token for the `developer` ServiceAccount.

## 3: Define a ClusterRole

Create a ClusterRole named `read-deploy-pod-clusterrole` with the following permissions:

- Verbs: get, list, watch
- Resources: pods, pods/log, deployments

## 4: Create a ClusterRoleBinding

Create a ClusterRoleBinding named `developer-read-deploy-pod` binding the `developer` ServiceAccount and the `read-deploy-pod-clusterrole` ClusterRole.

## 5: Generate a kubeconfig File

Generate a kubeconfig file for the developer.

**Note:** Users can choose the appropriate API version based on their Kubernetes environment.

---

*Good luck! We look forward to reviewing your solution.*
