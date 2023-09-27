# Kubernetes Cluster Administration

## Create service account developer

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: developer
```

## Manually create an API token for a ServiceAccount, referent: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#manually-create-an-api-token-for-a-serviceaccount
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: developer-secret
  annotations:
    kubernetes.io/service-account.name: developer
type: kubernetes.io/service-account-token
```

## Create ClusterRole
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: read-deploy-pod-clusterrole
rules:
  - verbs:
      - get
      - list
      - watch
    apiGroups:
      - ''
    resources:
      - pods
      - pods/log
  - verbs:
      - list
      - get
      - watch
    apiGroups:
      - apps
    resources:
      - deployments
```

## Create ClusterRoleBinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: developer-read-deploy-pod
subjects:
- kind: ServiceAccount
  name: developer
  namespace: default
roleRef:
  kind: ClusterRole
  name: read-deploy-pod-clusterrole
  apiGroup: rbac.authorization.k8s.io
```

## Finally, decode ca and token from developer-secret and paste them to the config file. Here is an example:
```yaml
apiVersion: v1
kind: Config
clusters:
  - name: demo-cluster
    cluster:
      server: https://x.x.x.x:6443/
      certificate-authority-data: >
        LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1EWXlOekEwTkRBeE0xb1hEVE15TURZeU5EQTBOREF4TTFvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBS1gvCmZmTW5NOHJKTWhBMHFkVm9RTkJBWEJFeXF4WjA3WnFwYUQ2bWx4ZFQxMEtMajJ6aHliekV6Q2NQTldCMzdsVncKbHZVZmlQelpjZ3lRN0krdWtXZDBUNWhRSjd4S0ozbTlFSnBJMDU2WEhKK2RKSXRuRlIybnUzY1gwdFIxREZMbQpUSG14eGlIRXdMdE1ESjdaNjdIcFR5Zk1JVjBwTXdMQXorRFNDMjNlLzludGJja045WExReUlobzc1MFZ2Y3lqCmJ5dmo0Y3dOSkFaNTU2Z21BdEY0a04ydVcxaWY2ZHJQZ2JQUU54Q2ZoMU1IVFB5QTBCdzVuQWVWNXg4UVB3eFIKcU0zWTZ3TVJwOGoxTFFFeGdPYTJuSVFNZ29tdlFSUEUzRzdSUW1WSWkvcDM2MkxRQldIVlJqT1lRTjd4cmlvQQpxKzRNOWRQaERja2Fxbm9LOVdFQ0F3RUFBYU5aTUZjd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZBaUlkWm9HU25wQ3QyTnBPbmtjMGN2US9nVFdNQlVHQTFVZEVRUU8KTUF5Q0NtdDFZbVZ5Ym1WMFpYTXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBRHdyTDVHVzdEMGdISk1ONC9mcgpIYU9wZktDR213VHVtVmdvZE9POHpFYzMwS1l0TXVsd2YxY1B4cGJlSDYzUExoaUNuckJldkJpNUdwTUNMVzd5CjQ5L2kyQ0FlN3RQWW01WENneVZxZHVma21RbU9URVoza2NuNzJnTWN4WGpNaW5abkc4MlFHWE5EUnZSQ3N3L1kKZ1JXSnhtLzc0ZVBQallqeDUrR05WZ1lkcHJHWmlkcFVCYVBtcXZFUzdUdTQ4TDVmRjRscGNDVkprSi9RblFxego1VEZpUzdGQ1ZXSW9PQ3ZhNzBGN0tLc1lSL3REeDhiV1RYMlkxeE5zMlpYd25sRlZLUy9neGhIMDVzV1NGK3JlCjJXL01qYUpoTWlBZ2FKUjJ2NDFzT0lsQUV1OXVmRC9RakN6QXU4VmNzTmMybzNoa1pGOEZNYUkxQ2VRaVI0WFEKeTk0PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
users:
  - name: developer
    user:
      token: >-
        eyJhbGciOiJSUzI1NiIsImtpZCI6IkxUOW1HdmxObnFEWDhlLTBTTkswaWJGcDkzZVVoWWZGRjJiNDhNZlpvY0UifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRldmVsb3Blci1zZWNyZXQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGV2ZWxvcGVyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiNzEyYTE0NTMtMzE5ZC00NzhlLWFjN2QtYzI1MzJmZGNlM2UzIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRlZmF1bHQ6ZGV2ZWxvcGVyIn0.sSIyyzCHzBCd4GVsjKdi_8bETitzG202ogFuT6EacUDbBWxYTpBv5s3otZOIANH5CPy2Ng6DAiTmq0mEzYTXL2j9AcNaFUYE7ydO3K2fpGwdLQ3qb5YbnZY9Y0mcIE3ShkyGISp0vIHr9RPlJv4QwbBSczDjvGikOKaVclWRdH4W0h5ngOfXINfRjLrJ9GQ7nUZED6jQYBA3CoEFFrbgEIBf8DUtSDF-IPxE2F8GlIBEHh0-gOHVTg4cef2NRsrUmI2uqy4JnlVeXzc4dbeToXQuBPHs7JmuQ1zzqxoXL-xsjvaH9sa9wan7Sjpgj9ZzTBClqUGJEbOxK2KnKxfG7Q
contexts:
  - name: demo-cluster-developer
    context:
      user: developer
      cluster: demo-cluster
```
