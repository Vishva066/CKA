# Creating a cluster with kubeadm

Here is a high level overview of creating a cluster using kubeadm

1. Disable swap

2. Enable kernel modules like overlay and br_netfilter

Overlay is used by container runtimes like **containerd and docker**.

It allows containers to use a layered file system (overlayfs) — which means:

- Base image files are read-only

- Changes by containers are stored as layers

Eg:

Let’s say you run a Node.js container.

- The base image has Node.js already.

- Your app writes logs or downloads files.

- Those changes go into a new layer — your container doesn’t touch the original image.


br_netfilter

This module allows Linux bridges to interact with the iptables firewall.

br_netfilter is used by **kube-proxy** to manage network traffic between pods and services.

By default, traffic inside the node doesn’t go through the firewall — the router just lets it pass freely.

That way, Kubernetes can apply rules like:

- Block or allow certain traffic

- NAT (change IPs for Services)

- Manage pod-to-pod, pod-to-service communication

3. Enable IP forwarding

This enables IP forwarding, which allows your node to route traffic from one pod to another — or from a pod to the internet.

```bash
net.ipv4.ip_forward = 1
```

4. Install container runtime (containerd or docker)

5. Install kubeadm, kubelet and kubctl

6. Initialize the cluster using kubeadm init

7. Join the worker nodes

8. Install a network plugin (CNI) like Calico or Flannel

## PKI certificates and requirements

Kubernetes requires PKI certificates for authentication over TLS.

If installed with kubeadm they are automatically generated and stored in the ****/etc/kubernetes/pki directory**.

Certificates are used for:

### Server Certificates

- API server certificate for api server endpoint

- server certificate for etcd server

- server certificate for kubelet

- Optional certificate for kube-proxy

### Client Certificates

- Client certificate for kubelet (used to authenticate to the API server as a client of the Kubernetes API)

- Client certificate for each API server (used to authenticate to etcd)

- Client certificate for the controller manager to securely communicate with the API server

- Client certificate for the scheduler to securely communicate with the API server

- Client certificates, one for each node, for kube-proxy to authenticate to the API server

### Certificate and their roles

| Certificate                | Key File                  | Cert File                  | Summary                                                                 |
|----------------------------|---------------------------|----------------------------|-------------------------------------------------------------------------|
| kubernetes-ca    | ca.key            | ca.crt             | Used to sign other certificates in the cluster.                        |
|etcd-ca    | etcd/ca.key       | etcd/ca.crt      | Used to sign etcd server and client certificates.                     |
|kube-apiserver-etcd-client   | apiserver-etcd-client.key  | apiserver-etcd-client.crt  | Used by the API server to authenticate to etcd.                        |
| kube-apiserver  | apiserver.key     | apiserver.crt    | Used by the API server to authenticate to clients.                    |
| kubelet-client | apiserver-kubelet-client.key | apiserver-kubelet-client.crt | Used by the API server to authenticate to kubelets.                   |
| kube-etcd-peer   | 	etcd/peer.key   | etcd/peer.crt  |  Used by etcd to authenticate to other etcd nodes. |
| kube-etcd-healthcheck-client   | etcd/healthcheck-client.key | etcd/healthcheck-client.crt | Used by etcd to authenticate to the API server. |


### Files and their roles

- admin.conf -> It stores credentials and connection info for kubectl to talk to the Kubernetes API server. It is created after running kubeadm init.

- kubelet.conf -> Kubelet uses this file to authenticate and communicate with the API server. Kubelet (the agent running on each node) reads this file for credentials and cluster details.

- controller-manager.conf -> Used by the kube-controller-manager component. It contains authentication details that the controller-manager needs to talk to the API server.

- scheduler.conf -> Used by the kube-scheduler component. Scheduler uses it to authenticate and schedule pods by talking to the API server.

#### Admin.conf

It has 3 major sections:

- Clusters – where the API server is

- Users – who is accessing the cluster

- Contexts – which user is accessing which cluster

Defines where the cluster is and how to securely connect to it. 

The base64 cert is the CA certificate for the cluster. (ca.crt)

Followed by the api server endpoint

**Users**

The users section lists all users already used to connect to a cluster.

- client-certificate contains a certificate for the user signed by the Kubernetes CA. This can be a file path or a Base64 string in the certificate PEM format.

- client-key contains the key that signed the client certificate.

**Context**

Maps a user to a cluster.

contexts:
- context:
    cluster: cluster.local
    user: kubernetes-admin
  name: kubernetes-admin@cluster.local
current-context: kubernetes-admin@cluster.local


## Decode a certificate

```bash
openssl x509 -in <cert-file> -text -noout
```

To view private key

```bash
sudo openssl rsa -in /etc/kubernetes/pki/apiserver.key -check
```

TO check which are certificates are all expiring execute the following command

```bash

kubeadm certs check-expiration
```

To renew the certificates execute this command

```bash
kubeadm certs renew -h

kubeadm certs renew <cert-name
```

## Kubeadm important commands

Initialize the cluster

```bash
kubeadm init
```

Initialize the cluster with specific pod network CIDR

```bash

kubeadm init --pod-network-cidr=<your-pod-network-cidr>
```

Join a worker node to the cluster

```bash
kubeadm join <your-cluster-endpoint> --token <your-token> --discovery-token-ca-cert-hash sha256:<your-hash>
```