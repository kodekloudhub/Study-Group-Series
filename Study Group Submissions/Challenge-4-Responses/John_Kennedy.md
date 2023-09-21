**Name**: John Kennedy

**Slack User ID**: U05D6BR916Z

**Email**: johnkdevops@gmail.com

---

## Solution

### Solution Details

# Question: Kubernetes Cluster Administration Task

## Task

### 1: Create a ServiceAccount

Using imperative commands to create & verify that the service account for the developer:
1. To create the service account, I ran k create sa developer. 
2. To verified the service account was created correctly, I ran k get sa developer
NAME        SECRETS   AGE
developer   0         3m16s

### 2: Generate an API Token

1. I searched for serviceaccount API Tokens on kubernetes docs.
Using imperative command to create API Token for developer service account:
2. To create API Token, I ran k create token developer.
eyJhbGciOiJSUzI1NiIsImtpZCI6IlB0T2gxRFR3bDZFdkxwZlQ3M1hyYjNxbXBIUFYyakc3aU1xS283a01id1EifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNjk1MzE4MDI1LCJpYXQiOjE2OTUzMTQ0MjUsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0Iiwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImRldmVsb3BlciIsInVpZCI6ImRiYWFhNDE0LWRlYmItNDc1YS1hMGIyLTg0ODIwYzhiMDVhOSJ9fSwibmJmIjoxNjk1MzE0NDI1LCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkZXZlbG9wZXIifQ.o3yLaVqmX75kwJobuxi70bZ8Ku-HxZ8DH5_j3-D-wmOhtx7vCFZdlQKUxXW5-J680unUGvbwjts4030QU0EGsSSmcut3_OxD18S3pCsnoTIcqzMZfbSRDDO5c7hUynToBNucH55v-CWJxcQFvRO2NL1DSAhuclNffC-D5L9aLN0PFmzghVBihUZebvTbigvF8qz18CFE6l7Vy1g2tT7AtOmO1vLFxmbKeFvUrR2EvdZMVAfbUGBPRiWX-cZ0yak6WHKHakD3dBbc-XNqFbePWV4BP6JGD7eRQVuTxoXOYpCzvXU0rmMxu8m-C3oAzowkQE8OLe1HV3PSiMK2K-PLcA


### 3: Define a ClusterRole

1. Search for pods/log in ClusterRoles in kubernetes docs.
Using imperative commands: get api-resources and create & verify that the ClusterRole for the developer
2. To verify correct names for resources, I ran k api-resources
Make sure I have correct resources: pods, deployments, pods/log
3. To create clusterrole, I ran k create clusterrole read-deploy-pod-clusterrole --verb=get,list,watch --resource=pods, pods/log, deployments --dry-run=client -o yaml
4. To verify the ClusterRole is created & permissions are correct, k describe clusterrole read-deploy-pod-clusterrole
Name:         read-deploy-pod-clusterrole
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources         Non-Resource URLs  Resource Names  Verbs
  ---------         -----------------  --------------  -----
  pods/log          []                 []              [get list watch]
  pods              []                 []              [get list watch]
  deployments.apps  []                 []              [get list watch]


### 4: Create a ClusterRoleBinding

Using imperative commands: create & verify ClusterRoleBinding for developer was created correctly.
1. To create clusterrolebinding, I ran k create clusterrolebinding developer-read-deploy-pod --clusterrole=read-deploy-pod-clusterrole --serviceaccount:default:developer --dry-run=client -o yaml
2. To verify the ClusterRoleBinding is created correctly by running k describe clusterrolebinding developer-read-deploy-pod
Name:         developer-read-deploy-pod
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  read-deploy-pod-clusterrole
Subjects:
  Kind            Name       Namespace
  ----            ----       ---------
  ServiceAccount  developer  default

3. To verify that ServiceAccount developer does have the correct permissions, I ran k auth can-i get po --as=system:serviceaccount:default:developer, got yes for all resources allowed, and when I ran k auth can-i create po --as=system:serviceaccount:default:developer, gave the back said, no.

# YAML File
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  creationTimestamp: null
  name: developer-read-deploy-pod
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: read-deploy-pod-clusterrole
subjects:
- kind: ServiceAccount
  name: developer
  namespace: default

### 5: Generate a kubeconfig File

1. Copied default kubeconfig file by running cp /root/.kube/config developer-config
2. Edited developer-config by running vi developer-config, made the changes shown below.
3. Verified that config is working by running k get po --kubeconfig developer-config.
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-5dd5756b68-c7zkz               1/1     Running   0          86m
coredns-5dd5756b68-ckx7f               1/1     Running   0          86m
etcd-controlplane                      1/1     Running   0          87m
kube-apiserver-controlplane            1/1     Running   0          87m
kube-controller-manager-controlplane   1/1     Running   0          87m
kube-proxy-k29l2                       1/1     Running   0          86m
kube-proxy-nzzqf                       1/1     Running   0          86m
kube-scheduler-controlplane            1/1     Running   0          87m

## YAML File
'developer-config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJSEp6UVJyV0MvVE13RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TXpBNU1qRXhOakExTXpaYUZ3MHpNekE1TVRneE5qRXdNelphTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUUMxM2dVREgvbGMrVFhNcFpjVGZNMWUvVW5pR2ZkZEpwd2FFWGZPZTR4dy9TTG1ycGRvUkh6MUt3d0sKOEZNRk9WVUlLSkRVT2t6dy9WUFJnSm43MndIU2ZPVTl0QjZqd0RhZTUyb2FrZDhzdmFKZUk0VFVTOGViM2dRRgpuV1RIT0JpcStIOU5Ca2V2R2I2MGszRHRHSEJmcG9OYVRxZzBOdC9yQUtPSnRsclhVWTVwOXhDSmYzbUZxQS94CmFEV0dYWENqTG1IemdNQXozcFJ4eWFFeHdsQzYzNW8xQy9tUzZseXF2b0dUajNjOXJBQjdXbHR3N2hKcGZqZzUKaDRRa2NiMk5wNVVXVzRnV3c1MDJBL0I3ZW9NNWtYYjVJcENOV05ZNVJvMXBrUnJ0WUVldlczNFF3WkVZcDNGOAoyM2hYUUhLYkpvNjd4ZlhBV1ZxNm5BVWxjbGdKQWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJTN1ZsUzFURzBBdWExRFpFWXBTL1VpckxGQ1BUQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQTFNN3JZMVFJcwpqaXlPcVowRFA4RllwcVpmdXZwZ2RZUHJaZFd6K25rTnl3NmJXZHhyd3BlR2tiWEd4RDVhSjYrMElJMnZkUTVhCnEwdXNWSEtNZ3FLdlBTT1VHMVNYZGgxSS91T0sreitHWFpxSDNOUWxXT1B6MWZ6bFQzWktJS25hazh5aHVTNDkKRWFHWEZnWFBxNFFtelBZd1lWVC9HSk8xdkJlVUlPMkw2TnN3VmtLRVBRZ2VLRTFtK0p5Zk13OHlnT3JDVmdERwpya2MxM3VuZFNOanZ3dlBDSjRjOGM1RU9IalJRajIzaHQ3eVpOd25uZEJXZVpGZFJEK3dkUG5RTTlTeWpUWDFOCnZaV2xxbWg5OVRPZG5ZNnVOdzNjdXd2SnBkT1A4WG96Uk5oditQYzM5aDFGQUh6dGRLMkNPK1ZXMkJjTXc4YksKMCtibzBCVGZxV0RmCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://controlplane:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURJVENDQWdtZ0F3SUJBZ0lJQ3AzZDE0VDVzcHd3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TXpBNU1qRXhOakExTXpaYUZ3MHlOREE1TWpBeE5qRXdNemxhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQW9xUGdCMnZWSDRQTDYrMk0KZGR3YUlvY21jUlJRQlJTcEFua0RFdVVUaWtEbjY1c0dWbUNJMXdlamw5YzRLcU1HVGx4UHBBakpKemdyZS9DTAptVmtlTDdhSDN0RVVJaGpORUgwQTJLNitrYmFib3BLNXo4V3JJS0xicDBqSWR1eUU2YjI3bDd3VU52MWx1UmtFCkpxY0swMWVIaWVvUFIxaG44a2hBK05BSE1BQ3ZSSTNybjNVd1U2THpZUWltdS9lV0hET0RmUjc2OUZ4alFGSTkKaHJMLzhlMDlMaDBHa0RXVFozRmNSMXdDZGovaUtCRUN6UFdZUDBRTlNKamtTSDZuMUZ0aDQ4ZlI0YnQrVTF5TgpZRWFRTE1rKzdJSnBtUEFhNURkd0x4OXFka2s4VVo1aUhsWlJnamtOekVoMUdib2MrdmNCWXVpVm9ZN0tHZ3ZsCktkc2Fxd0lEQVFBQm8xWXdWREFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RBWURWUjBUQVFIL0JBSXdBREFmQmdOVkhTTUVHREFXZ0JTN1ZsUzFURzBBdWExRFpFWXBTL1VpckxGQwpQVEFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBRlY4dnh3OTdsT2lCM2crZTJwYU1Ya0N1RHpLUERxNTJxQllYCkF6YVdqMXhnSGp3bWtUa2k2TmRVUmhUdFNjUFVzWHZnNi9QOFpuaEd1UGkyU3d0dzY5SUMxU3VrMG90cndKaTcKMFdaaHgyM2FNZ0dvNVRJZnNEQTJqUWRQM3A0bUpUNElIVHkzanhFbEdHVS9BV3oreTM1UnlyQk4zSDcvNDlkQQo5OHFpWDU1UUppYlJBRXorU3V3b0pNQk1VaVRHdlgzZUw2SGgyd25zbXdHMU8xeTFjdm12dmVNWDZpN01YdXpBCjN5ZDlJa2lpdDNPVkdwNVU1aDVVdUtUS0MvN2ZmTzViNGtSTDdrNW5nc1RvWEt6bWtUVjlDMy81ZjhyWXo3YU8KVGYwaytBOFBESks3R3R3Q24wSHJvWE5lK0locHpmMENZRHQxdmxnNGhZblFoVTlZc1E9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    
Now, the developer-config file looks like this:
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJSEp6UVJyV0MvVE13RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TXpBNU1qRXhOakExTXpaYUZ3MHpNekE1TVRneE5qRXdNelphTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUUMxM2dVREgvbGMrVFhNcFpjVGZNMWUvVW5pR2ZkZEpwd2FFWGZPZTR4dy9TTG1ycGRvUkh6MUt3d0sKOEZNRk9WVUlLSkRVT2t6dy9WUFJnSm43MndIU2ZPVTl0QjZqd0RhZTUyb2FrZDhzdmFKZUk0VFVTOGViM2dRRgpuV1RIT0JpcStIOU5Ca2V2R2I2MGszRHRHSEJmcG9OYVRxZzBOdC9yQUtPSnRsclhVWTVwOXhDSmYzbUZxQS94CmFEV0dYWENqTG1IemdNQXozcFJ4eWFFeHdsQzYzNW8xQy9tUzZseXF2b0dUajNjOXJBQjdXbHR3N2hKcGZqZzUKaDRRa2NiMk5wNVVXVzRnV3c1MDJBL0I3ZW9NNWtYYjVJcENOV05ZNVJvMXBrUnJ0WUVldlczNFF3WkVZcDNGOAoyM2hYUUhLYkpvNjd4ZlhBV1ZxNm5BVWxjbGdKQWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJTN1ZsUzFURzBBdWExRFpFWXBTL1VpckxGQ1BUQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQTFNN3JZMVFJcwpqaXlPcVowRFA4RllwcVpmdXZwZ2RZUHJaZFd6K25rTnl3NmJXZHhyd3BlR2tiWEd4RDVhSjYrMElJMnZkUTVhCnEwdXNWSEtNZ3FLdlBTT1VHMVNYZGgxSS91T0sreitHWFpxSDNOUWxXT1B6MWZ6bFQzWktJS25hazh5aHVTNDkKRWFHWEZnWFBxNFFtelBZd1lWVC9HSk8xdkJlVUlPMkw2TnN3VmtLRVBRZ2VLRTFtK0p5Zk13OHlnT3JDVmdERwpya2MxM3VuZFNOanZ3dlBDSjRjOGM1RU9IalJRajIzaHQ3eVpOd25uZEJXZVpGZFJEK3dkUG5RTTlTeWpUWDFOCnZaV2xxbWg5OVRPZG5ZNnVOdzNjdXd2SnBkT1A4WG96Uk5oditQYzM5aDFGQUh6dGRLMkNPK1ZXMkJjTXc4YksKMCtibzBCVGZxV0RmCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://controlplane:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: developer 
  name: developer-context
current-context: developer-context
kind: Config
preferences: {}
users:
- name: developer
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6IlB0T2gxRFR3bDZFdkxwZlQ3M1hyYjNxbXBIUFYyakc3aU1xS283a01id1EifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNjk1MzE4MDI1LCJpYXQiOjE2OTUzMTQ0MjUsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0Iiwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImRldmVsb3BlciIsInVpZCI6ImRiYWFhNDE0LWRlYmItNDc1YS1hMGIyLTg0ODIwYzhiMDVhOSJ9fSwibmJmIjoxNjk1MzE0NDI1LCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkZXZlbG9wZXIifQ.o3yLaVqmX75kwJobuxi70bZ8Ku-HxZ8DH5_j3-D-wmOhtx7vCFZdlQKUxXW5-J680unUGvbwjts4030QU0EGsSSmcut3_OxD18S3pCsnoTIcqzMZfbSRDDO5c7hUynToBNucH55v-CWJxcQFvRO2NL1DSAhuclNffC-D5L9aLN0PFmzghVBihUZebvTbigvF8qz18CFE6l7Vy1g2tT7AtOmO1vLFxmbKeFvUrR2EvdZMVAfbUGBPRiWX-cZ0yak6WHKHakD3dBbc-XNqFbePWV4BP6JGD7eRQVuTxoXOYpCzvXU0rmMxu8m-C3oAzowkQE8OLe1HV3PSiMK2K-PLcA

