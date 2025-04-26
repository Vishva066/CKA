# High Availability Cluster

Highly available kubernetes cluster can be set up in two ways 

- Creating multiple control pane nodes in the same server. (Stacked ETCD)
- Creating an external etcd cluster. (External ETCD).

## Stacked ETCD

Three or more control pane nodes. Having an odd number of control pane nodes help in leader selection. 

Kubernetes control plane components like etcd (the key-value store) need to agree on changes — like adding a new node or updating configuration.
This is based on majority so if there are 2 nodes even though 1 node goes down it needs more than 50% but since there is only one node it cannot agree on the desired state. That is why odd no of control pane nodes is recommended. 

1. Create a load balncer for the kube-api-server

- In a cloud environment you should place your control plane nodes behind a TCP forwarding load balancer. 

- The load balancer must be able to communicate with all control plane nodes on the apiserver port. It must also allow incoming traffic on its listening port.

2. Initializing the control pane nodes

```bash

sudo kubeadm init --control-plane-endpoint "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT" --upload-certs
```

The upload-certs flag is used to upload the certificates that should be shared across all the control-plane instances to the cluster.
