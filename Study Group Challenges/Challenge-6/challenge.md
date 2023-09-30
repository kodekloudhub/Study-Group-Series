### Kubernetes Challenge: Ingress and Networking

Imagine you have a development cluster with Nginx Ingress already installed. Your objective is to configure a single domain to route traffic to two different services. Since this is a development environment, you have the flexibility to employ virtual domains. Specifically, you need to map the virtual domain `cka.kodekloud.local` to your IP node.

1. **Create a `wear-deployment`:**
   - Name: `wear-deployment`
   - Image: `kodekloud/ecommerce:apparels`
   - Port: `8080`
   - Expose this deployment.

2. **Create a `video-deployment`:**
   - Name: `video-deployment`
   - Image: `kodekloud/ecommerce:video`
   - Port: `8080`
   - Expose this deployment.

3. **Create an Ingress:**
   - Name: `ecommerce-ingress`
   - Annotations: `nginx.ingress.kubernetes.io/rewrite-target: /`
   - Host: `cka.kodekloud.local`
   - Paths: 
     - `/wear` routes to service `wear`
     - `/video` routes to service `video`

4. **Point virtual domain `cka.kodekloud.local` to your cluster and test it.**

5. **Enable pods to access the domain `cka.kodekloud.local`.**
   - Describe the steps you would take to enable this functionality and verify it.

---

Good luck!
