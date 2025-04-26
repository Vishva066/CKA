# Resource Constraints

Whenver we are scheduling a container we can specify resources for that container/pod. These resources are usually **CPU and RAM** but there are others also.

When you specify the resource request for containers in a Pod, the kube-scheduler uses this information to decide which node to place the Pod on.

## CPU and Memory Resource Units

Fractional Requests are allowed for CPU but not for Memory. CPU is measured in units of cores, and memory is measured in bytes. The following units are supported:

Unit | Description
-----|----------------
m    | milli CPU (1/1000 of a core)
K    | KiloBytes
M    | MegaBytes
G    | GigaBytes
T    | TeraBytes


Eg:

CPU 0.1 is equivalent to the expression 100m

CPU 1 is equivalent to the expression 1000m

**Note: Kubernetes doesn't allow you to specify CPU resources with a precision finer than 1m or 0.001 CPU. To avoid accidentally using an invalid CPU quantity, it's useful to specify CPU units using the milliCPU form instead of the decimal form when using less than 1 CPU unit.**

Kubernetes doesn’t allow less than 1m (millicpu). That’s the smallest unit.

You can express memory as a plain integer or as a fixed-point number using one of these quantity suffixes: E, P, T, G, M, k. You can also use the power-of-two equivalents: Ei, Pi, Ti, Gi, Mi, Ki.

Eg

128974848, 129e6, 129M,  128974848000m, 123Mi

## Container Resource Requests and Limits

Specifying requests and limits to the containers

Eg

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: log-aggregator
    image: images.my-company.example/log-aggregator:v6
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

## Pod Resoruces Requests and Limits

Specifying requests and limits to the pods

Eg

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-resources-demo
  namespace: pod-resources-example
spec:
  resources:
    limits:
      cpu: "1"
      memory: "200Mi"
    requests:
      cpu: "1"
      memory: "100Mi"
  containers:
  - name: pod-resources-demo-ctr-1
    image: nginx
    resources:
      limits:
        cpu: "0.5"
        memory: "100Mi"
      requests:
        cpu: "0.5"
        memory: "50Mi"
  - name: pod-resources-demo-ctr-2
    image: fedora
    command:
    - sleep
    - inf 
```
Here the difference is in the pod resouces section. We have defined the resources and limits in the spec section of the pod.

But there we have defined it under the container section. So the resources and limits defined in the pod spec will be applied to all the containers in the pod. The resources and limits defined in the container spec will be applied to that specific container only.

