# Pod Priority

Pod priority is a Kubernetes scheduling feature that allows Kubernetes to make scheduling decisions comparing other pods based on priority number

## Pod Preemeption

The pod preemption feature allows Kubernetes to preempt (evict) lower-priority pods from nodes when higher-priority pods are in the scheduling queue and no node resources are available.

## Pod Priority Class

You can set a priority for a Pod using the PriorityClass object (non-namespaced) with a Value.

The value determines the priority. It can be 1,000,000,000 (one billion) or lower. Larger the number, the higher the priority.

If you don't want the priority class to preempt the pods, you can set PreemptionPolicy: Never. By default, Priorityclasss use PreemptLowerPriority policy.

Priority Class example

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority-apps
value: 1000000
preemptionPolicy: PreemptLowerPriority
globalDefault: false
description: "Mission Critical apps."
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: dev
spec:
  containers:
  - name: web
    image: nginx:latest
    imagePullPolicy: IfNotPresent
  priorityClassName: high-priority-apps
```

### How do you safeguard system-critical pods from preemption?

Here we have to set high priority classes

Well, there are two default high-priority classes set by Kubernetes

1. system-node-critical: This class has a value of 2000001000. Static pods Pods like etcd, kube-apiserver, kube-scheduler and Controller manager use this priority class.

2. system-cluster-critical: This class has a value of 2000000000. Addon Pods like coredns, calico controller, metrics server, etc use this Priority class.

## How does Pod Priority and Preemption work

1. If a pod is deployed with PriorityClassName, the priority admission controller gets the priority value using the PriorityClassName value.

2. If there are many pods in the scheduling queue, the scheduler arranges the scheduling order based on priority. Meaning, the scheduler places the high-priority pod ahead of low priority pods

3. Now, if there are no nodes available with resources to accommodate a higher-priority pod, the preemption logic kicks in.

4. The scheduler preempts (evicts) low priority pod from a node where it can schedule the higher-priority pod. The evicted pod gets a graceful default termination time of 30 seconds. If pods have terminationGracePeriodSeconds set for preStop container Lifecycle Hooks, it overrides the default 30 seconds.

5. owever, if for some reason, the scheduling requirements are not met, the scheduler goes ahead with scheduling the lower-priority pods.

### References

1. https://devopscube.com/pod-priorityclass-preemption/
