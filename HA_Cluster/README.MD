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

3. Apply the CNI plugin

4. Join the control plane nodes

```bash

sudo kubeadm join lb.example.com:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH> --control-plane --certificate-key <CERTIFICATE_KEY>

```

## External ETCD

1. Make sure you have multiple instance of etcd running

2. Create a file called kubeadm-config.yaml

```yaml
apiVersion: kubeadm.k8s.io/v1beta4
kind: ClusterConfiguration
kubernetesVersion: stable
controlPlaneEndpoint: "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT" # change this (see below)
etcd:
  external:
    endpoints:
      - https://ETCD_0_IP:2379 # change ETCD_0_IP appropriately
      - https://ETCD_1_IP:2379 # change ETCD_1_IP appropriately
      - https://ETCD_2_IP:2379 # change ETCD_2_IP appropriately
    caFile: /etc/kubernetes/pki/etcd/ca.crt
    certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
    keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
```

3. Run the kubeadm init command with the config file

```bash
sudo kubeadm init --config kubeadm-config.yaml --upload-certs
```


## Example Questions

1. You are tasked to initialize a highly available Kubernetes control plane with kubeadm. You must specify a load balancer address lb.example.com on port 6443.
Which command will you use?

```bash

sudo kubeadm init --control-plane-endpoint "lb.example.com:6443" --upload-certs
```

2. After setting up the first control-plane node, how do you join a second control-plane node?

```bash
sudo kubeadm join lb.example.com:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH> --control-plane --certificate-key <CERTIFICATE_KEY>
```

3. Why do we use --upload-certs during kubeadm init?

It uploads the control-plane certificates to a kubeadm-generated secret (kubeadm-certs) so that additional control-plane nodes can pull certificates easily.

This is required for setting up multiple control-plane nodes.

4. Which file do you edit to configure kube-apiserver to advertise the load balancer address?

/etc/kubernetes/manifests/kube-apiserver.yaml

Set --advertise-address properly if needed.

5. What is the purpose of the --certificate-key when joining a second control plane node?


To explicitly specify the key used to encrypt the control-plane certificates. This is required for the new control-plane node to decrypt and use the certificates uploaded by the first control-plane node.

6. Your load balancer forwards traffic to 3 control-plane nodes. Which component inside the kubeadm config file (ClusterConfiguration) will you set to define the load balancer endpoint?

**controlPlaneEndpoint**