# KUBE PROXY

Kube-proxy is a network component in Kubernetes that manages network rules on nodes. It ensures that network traffic is properly routed to the correct Pods in a cluster. Kube-proxy uses iptables or IPVS to handle traffic routing and load balancing for services.

The Kubernetes network proxy runs on each node.

### Example: Kube-proxy in Action

1. Create a simple deployment:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      replicas: 2
      selector:
         matchLabels:
            app: nginx
      template:
         metadata:
            labels:
              app: nginx
         spec:
            containers:
            - name: nginx
              image: nginx:1.21
              ports:
              - containerPort: 80
    ```

2. Expose the deployment as a service:
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-service
    spec:
      selector:
         app: nginx
      ports:
      - protocol: TCP
         port: 80
         targetPort: 80
      type: ClusterIP
    ```

3. Access the service:
    Use the `nginx-service` ClusterIP to route traffic to the Nginx Pods. Kube-proxy ensures the traffic is load-balanced between the Pods.

This demonstrates how Kube-proxy handles routing and load balancing for services in Kubernetes.

### References

1. https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/?ref=devopscube.com

