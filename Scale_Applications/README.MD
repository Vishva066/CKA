# Scaling the application

In order to reduce the no of replicas execute this command

```bash
kubectl scale deployment/my-nginx --replicas=1
```

If you want to have min and max no of pods based on the load, you can setup autoscale using this method

```bash
kubectl autoscale deployment/my-nginx --min=1 --max=3
```

## kubectl scale

Set a new size for a deployment, replica set, replication controller, or stateful set.

Scale also allows users to specify one or more preconditions for the scale action.

**Syntax**

```bash
kubectl scale [--resource-version=version] [--current-replicas=count] --replicas=COUNT (-f FILENAME | TYPE NAME)

```

**Examples**

```bash
# Scale a replica set named 'foo' to 3
  kubectl scale --replicas=3 rs/foo
  
  # Scale a resource identified by type and name specified in "foo.yaml" to 3
  kubectl scale --replicas=3 -f foo.yaml
  
  # If the deployment named mysql's current size is 2, scale mysql to 3
  kubectl scale --current-replicas=2 --replicas=3 deployment/mysql
  
  # Scale multiple replication controllers
  kubectl scale --replicas=5 rc/example1 rc/example2 rc/example3
  
  # Scale stateful set named 'web' to 3
  kubectl scale --replicas=3 statefulset/web

```

## kubectl autoscale

Creates an autoscaler that automatically chooses and sets the number of pods that run in a Kubernetes cluster.


An autoscaler can automatically increase or decrease number of pods deployed within the system as needed.

**Syntax**

```bash
kubectl autoscale (-f FILENAME | TYPE NAME | TYPE/NAME) [--min=MINPODS] --max=MAXPODS [--cpu-percent=CPU]
```

**Examples**

```bash
  # Auto scale a deployment "foo", with the number of pods between 2 and 10, no target CPU utilization specified so a default autoscaling policy will be used
  kubectl autoscale deployment foo --min=2 --max=10
  
  # Auto scale a replication controller "foo", with the number of pods between 1 and 5, target CPU utilization at 80%
  kubectl autoscale rc foo --max=5 --cpu-percent=80
```

**Working**

- You currently have 1 pod.

- Your app gets a surge in traffic, and that pod reaches 160% CPU.

- The Horizontal Pod Autoscaler (HPA) kicks in and says, “Hmm, the average CPU is too high.”

- It adds a 2nd pod → checks again → still high → adds 3rd... and so on.

- It stops at 5 pods, even if CPU is still above 80%, because that’s the maximum limit you allowed.

## Horizontal Pod Autoscaler

Horizontal scaling means that the response to increased load is to deploy more Pods.

This is different from vertical scaling, which for Kubernetes would mean assigning more resources (for example: memory or CPU) to the Pods that are already running for the workload.

If the load decreases, and the number of Pods is above the configured minimum, the HorizontalPodAutoscaler instructs the workload resource (the Deployment, StatefulSet, or other similar resource) to scale back down.

Horizontal pod autoscaling does not apply to objects that can't be scaled (for example: a DaemonSet.)

**Working of Horizontal Pod Autoscaler**

- HPA watches metrics like CPU, memory, or custom metrics.

- It automatically adjusts the number of pods to keep the metric around a target value (e.g. 80% CPU).

Eg of HPA

```bash
kubectl create deployment nginx --image=nginx

kubectl patch deployment nginx --patch '{
  "spec": {
    "template": {
      "spec": {
        "containers": [{
          "name": "nginx",
          "resources": {
            "requests": {
              "cpu": "100m"
            }
          }
        }]
      }
    }
  }
}'

```

```bash
kubectl autoscale deployment nginx --cpu-percent=50 --min=1 --max=5
```

Check HPA status

```bash
kubectl get hpa
```


### References

1. https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/