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

5. Install kubeadm, kubelet and kubectl

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

## Kubeadm init

On running **kubeadm init** these are the following steps performed

- preflight                     Run pre-flight checks
- certs                         Certificate generation
  - /ca                           Generate the self-signed Kubernetes CA to provision identities for other Kubernetes components
  - /apiserver                    Generate the certificate for serving the Kubernetes API
  - /apiserver-kubelet-client     Generate the certificate for the API server to connect to kubelet
  - /front-proxy-ca               Generate the self-signed CA to provision identities for front proxy
  - /front-proxy-client           Generate the certificate for the front proxy client
  - /etcd-ca                      Generate the self-signed CA to provision identities for etcd
  - /etcd-server                  Generate the certificate for serving etcd
  - /etcd-peer                    Generate the certificate for etcd nodes to communicate with each other
  - /etcd-healthcheck-client      Generate the certificate for liveness probes to healthcheck etcd
  - /apiserver-etcd-client        Generate the certificate the apiserver uses to access etcd
  - /sa                           Generate a private key for signing service account tokens along with its public key
- kubeconfig                    Generate all kubeconfig files necessary to establish the control plane and the admin kubeconfig file
  - /admin                        Generate a kubeconfig file for the admin to use and for kubeadm itself
  - /super-admin                  Generate a kubeconfig file for the super-admin
  - /kubelet                      Generate a kubeconfig file for the kubelet to use *only* for cluster bootstrapping purposes
  - /controller-manager           Generate a kubeconfig file for the controller manager to use
  - /scheduler                    Generate a kubeconfig file for the scheduler to use
- etcd                          Generate static Pod manifest file for local etcd
  - /local                        Generate the static Pod manifest file for a local, single-node local etcd instance
- control-plane                 Generate all static Pod manifest files necessary to establish the control plane
  - /apiserver                    Generates the kube-apiserver static Pod manifest
  - /controller-manager           Generates the kube-controller-manager static Pod manifest
  - /scheduler                    Generates the kube-scheduler static Pod manifest
- kubelet-start                 Write kubelet settings and (re)start the kubelet
- upload-config                 Upload the kubeadm and kubelet configuration to a ConfigMap
  - /kubeadm                      Upload the kubeadm ClusterConfiguration to a ConfigMap
  - /kubelet                      Upload the kubelet component config to a ConfigMap
- upload-certs                  Upload certificates to kubeadm-certs
- mark-control-plane            Mark a node as a control-plane
- bootstrap-token               Generates bootstrap tokens used to join a node to a cluster
- kubelet-finalize              Updates settings relevant to the kubelet after TLS bootstrap
  - /enable-client-cert-rotation  Enable kubelet client certificate rotation
- addon                         Install required addons for passing conformance tests
  - /coredns                      Install the CoreDNS addon to a Kubernetes cluster
  - /kube-proxy                   Install the kube-proxy addon to a Kubernetes cluster
- show-join-command             Show the join command for control-plane and worker node

To execute steps by steps use this following command

```bash
Available Commands:
  phase       Use this command to invoke single phase of the "init" workflow
```

## Some common kubeadm certificate based questions

1. Initialize a Kubernetes control plane with kubeadm using a specific Pod network CIDR 192.168.0.0/16.

```bash
kubeadm init --pod-network-cidr=192.168.0.0/16
```

2. Generate a new certificate key to use for uploading control plane certificates.

```bash
kubeadm init phase upload-certs
```

3. Renew all certificates for the Kubernetes control plane

```bash
kubeadm certs renew all
```

4. Check the expiration dates of Kubernetes certificates.

```bash
kubeadm certs check-expiration
```

5. Create a kubeconfig file manually for an admin user if it got deleted.

```bash
kubeadm init phase kubeconfig admin
```

6. Initialize a Kubernetes cluster with kubeadm, setting the control plane endpoint to loadbalancer.example.com:6443 and uploading certificates.

```bash
kubeadm init --control-plane-endpoint=loadbalancer.example.com:6443
```

7. Renew only the apiserver certificate without touching others.

```bash
kubeadm certs renew -h

kubeeadm certs renew apiserver
```

8. Check how many days are left before the Kubernetes certificates expire.

```bash
openssl x509 -in /etc/kubernetes/pki/ca.crt -text -noout | grep 'Not After'
```

9. After a node crash, regenerate the controller-manager.conf kubeconfig file.

```bash
kubeadm init phase kubeconfig controller-manager
```

10. During kubeadm init, how can you set a specific Kubernetes version, for example, v1.30.0?

```bash
kubeadm init --kubernetes-version v1.30.0
```

## References

- https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/