# Ugrading the kubernetes cluster

First decide how you want to upgrade the cluster. Is it going to be a minor upgrade or a bug fix upgrade?

Minor upgrade represents:

v1.31.0 -> v1.32.0

Bug fix upgrade represents:

v1.31.0 -> v1.31.1

Follow the following steps if it is a minor upgrade.

**If it is minor upgrade first we have to change the kubernetes package repository accordingly.**

```bash
sudo nano /etc/apt/sources.list.d/kubernetes.list

#Change the version in the URL to the next available minor release, for example:

deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /
```

1. Decide which version to upgrade to by executing the following commands

```bash
sudo apt update

sudo apt-cache madison kubeadm
```

## Upgrading the control pane nodes

1. Upgrade kubeadm

Edit the kubeadm version according to your recommended version

```bash
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.32.x-*' && \
sudo apt-mark hold kubeadm
```

When a package is on hold, it will not be upgraded automatically by the package manager.

Check if the expected version has been downloaded

```bash
kubeadm version
```

Verify the upgrade plan using this command

```bash
sudo kubeadm upgrade plan
```

This command checks that your cluster can be upgraded, and fetches the versions you can upgrade to. It also shows a table with the component config version states.

Eg:

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   NODE      CURRENT   TARGET
kubelet     master    v1.31.7   v1.32.3
kubelet     worker1   v1.32.3   v1.32.3
kubelet     worker4   v1.32.3   v1.32.3

Upgrade to the latest stable version:

COMPONENT                 NODE      CURRENT    TARGET
kube-apiserver            master    v1.31.7    v1.32.3
kube-controller-manager   master    v1.31.7    v1.32.3
kube-scheduler            master    v1.31.7    v1.32.3
kube-proxy                          1.31.7     v1.32.3
CoreDNS                             v1.11.3    v1.11.3
etcd                      master    3.5.15-0   3.5.16-0

```bash
sudo kubeadm upgrade apply v1.32.x
```

IF you have any other control pane nodes execute the following command 

```bash
sudo kubeadm upgrade node
```

## Worker nodes upgradation

Prepare the node for maintenance by marking it unschedulable and evicting the workloads:

```bash
kubectl drain <node-to-drain> --ignore-daemonsets
```

Install kubeadm on worker nodes

```bash
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.32.x-*' && \
sudo apt-mark hold kubeadm```
```

Then run thsi command to upgrade the node

```bash
kubeadm upgrade node
```
## Upgrade kubelet and kubectl

```bash
# replace x in 1.32.x-* with the latest patch version
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.32.x-*' kubectl='1.32.x-*' && \
sudo apt-mark hold kubelet kubectl
```

Change the version accordingly to previous upgrade

Restart the kubelet using this command

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

Uncordon the node using this command

(Uncordon means making it again to schedulable from mainatainance)

```bash
kubectl uncordon <node-to-uncordon>
```
Here <node-to-uncordon> is the name of the node you want to uncordon in this case the control plane node

## Upgrade the worker nodes

The worker nodes also follows the same steps as the control pane.

We need to make sure not to upgrade all the worker nodes at the same time so that there will be no downtime.

Change the k8s package repo

```bash
sudo nano /etc/apt/sources.list.d/kubernetes.list

#Change the version in the URL to the next available minor release, for example:

deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /
```

1. Upgrade kubeadm

```bash
# replace x in 1.32.x-* with the latest patch version
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.32.x-*' && \
sudo apt-mark hold kubeadm
```

2. Upgrade the worker nodes

```bash
sudo kubeadm upgrade node
```

3. Drain the node

```bash
kubectl drain <node-to-drain> --ignore-daemonsets
```

4. Upgrade kubectl and kubelet

```bash
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.32.x-*' kubectl='1.32.x-*' && \
sudo apt-mark hold kubelet kubectl
```

5. Restart kubelet using this command

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

6. Uncordon the node using this command

```bash
kubectl uncordon <node-to-uncordon>
```


## References

- https://k21academy.com/docker-kubernetes/k8s-cluster-upgrade-step-by-step/

- https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

