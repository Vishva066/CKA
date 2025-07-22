# TroubleShooting Cluster

To get the overall health of the cluster execute the following command:

```bash
kubectl cluster-info dump
```

We can also describe the nodes just like pod 

```bash

#Syntax

kubectl describe node <node-name>

#Example

kubectl describe node node1

# To get it in yaml format 

kubectl describe node node1 -o yaml
```

## Static Pod Log file Locations

**Master Node**

- Apiserver -> /var/log/container/kube-apiserver.log

- Scheduler -> /var/log/container/kube-scheduler.log

- Controller Mangaer -> /var/log/container/kube-controller-manager.log

**Worker Node and Master Node**

- Kubelet -> /var/log/container/kubelet.log

- Kube Proxy -> /var/log/container/kube-proxy.log

Can also use journalctl for logging

## Journalctl logging

Journalctl can be used to view system level logs (It can be used to see the logs of journalctl)

```bash
journalctl -u kubelet
```