# Weekly Challenge: Service Availability During Deployment

## Task: Ensuring Service Availability During and After Deployments

**Objective:** Propose a solution to ensure high availability of the service during and after deployments, considering the following requirements:

1. Minimize the occurrence of 503 - Service Unavailable errors during deployments.
2. Ensure that the service is only considered ready when all its required dependencies are also ready.

### Proposed Solution

To meet these requirements, we can implement the following steps:

1. **ReadinessProbe Configuration:** Configure a readinessProbe for the pod to determine when the service is ready to accept traffic. This probe can be customized based on your specific service requirements. For example, you can check if the service's critical endpoints are responsive, and all dependencies are ready. Here's an example of a readinessProbe configuration:

```yaml
readinessProbe:
  httpGet:
    path: /healthz
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5
```

1. **Rolling Updates:** Ensure that the deployment strategy type is set to RollingUpdate. This ensures that Kubernetes will gradually update the pods during deployments, reducing the risk of a sudden surge in traffic to the new pods that might not be fully ready.

    Here's an example of how to set the deployment strategy type to RollingUpdate in a deployment manifest:



```
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 25%
    maxSurge: 25%
```

1. **Dependency Checks:** Implement checks within your service code to verify that all required dependencies (Database, Kafka, Redis, etc.) are reachable and ready before marking the service as ready. You can use health checks or custom logic to achieve this.

2. **Monitoring and Alerts:** Set up monitoring for your service and dependencies. Use tools like Prometheus and Grafana to collect metrics and set alerts for any anomalies or service unavailability.

    By following these steps, you can ensure high availability of your service during and after deployments while minimizing the occurrence of 503 - Service Unavailable errors. 

    Additionally, this approach promotes a smoother transition during updates and reduces the impact on end-users.