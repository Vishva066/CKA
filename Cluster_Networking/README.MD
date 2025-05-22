# Cluster Networking

Networking areas:

1. Highly-coupled container-to-container communications: this is solved by Pods and localhost communications.

2. Pod-to-Pod communications: this is the primary focus of this document.

3. Pod-to-Service communications: this is covered by Services.

4. External-to-Service communications: this is also covered by Services.

## IP Address Management

Kubernetes clusters require to allocate non-overlapping IP addresses for Pods, Services and Nodes, from a range of available addresses configured in the following components

- The network plugin is configured to assign IP addresses to Pods.

- The kube-apiserver is configured to assign IP addresses to Services.

- The kubelet or the cloud-controller-manager is configured to assign IP addresses to Nodes.

## Cluster Networking Sample Questions

1. Create a Pod named hostnet-pod using the nginx image that runs in the host network namespace.

```bash
kubectl run hostnet-pod --image=nginx --overrides='
{
  "spec": {
    "hostNetwork": true,
    "containers": [{
      "name": "nginx",
      "image": "nginx"
    }]
  }
}' --restart=Never

```
hostNetwork: true makes the pod share the node’s network namespace (i.e., pod uses node IP and ports directly).

2. 