# Deployment

Kubernetes deployment Yaml contains the following main specifications.

1. apiVersion
This specifies the API version of the Kubernetes deployment object. It varies between each Kubernetes version.

- Alpha (scalingpolicy.kope.io/v1alpha1)

- Beta (batch/v1beta1)

- Stable (apps/v1)

2. Kind

Kind describes the type of the object/resource to be created. (Deployment)

3. metadata
It is a set of data to uniquely identify a Kubernetes object. Following are the key metadata that can be added to an object.

- labels -> Key-value pairs primarily used to group and categorize deployment object.
- name -> Name of the deployment object.
- namespace -> Namespace in which the deployment is created.
- annotations -> key-value pairs like labels, however, used for different use cases.

4. spec
Here we declare the desired state and characteristics of the object we want to have. Spec has three important subfields.

- replicas -> Number of replicas to be running all the time.

- selector ->  It defines the labels that match the pods for the deployments to manage.

```yaml
selector:
    matchLabels:
      app: nginx
```

- Template ->  It has its own metadata and spec. Spec will have all the container information a pod should have. Container image info, port information, ENV variables, command arguments etc. Example

```yaml
template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - image: nginx
          name: nginx
```

## Deployment Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
  namespace: deployment-demo
  annotations:
    monitoring: "true"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: "2Gi"
            cpu: "1000m"
          requests: 
            memory: "1Gi"
            cpu: "500m"
```

**Service Example**

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx
  namespace: deployment-demo
spec:
  ports:
  - nodePort: 30500
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: NodePort
```

## Kubectl commands

Create a deployment using this command

```bash
kubectl create deploy <name> --image=<image> --replicas=<no_of_replicas> --namespace=<namespace>

k create deploy <name> --image=<image> --replicas=<no_of_replicas> --namespace=<namespace>
```

Create a deployment with arguments using this commands

```bash
k create deploy <name> --image=<name> -- sleep 300 #with command arguments
```

To scale a particular kubernetes deployment use this command

```bash
k scale deploy <name> --replicas=4 # scale up or down
```

Dry run -> Dry run refers to the process of testing the deployment without actually deploying it. It is useful for validating the configuration and ensuring that everything is set up correctly before applying changes to the cluster.

--dry-run=client
The validation is done locally by your machine.

It does not contact the API server.

Fast, but it might miss cluster-specific validations.

✅ Use when: you're just testing manifest syntax.

2. --dry-run=server
Sends the request to the API server with validation, but the server doesn’t save anything.

It checks against live cluster settings.

```bash
k create deploy web-deployment \
     --image=nginx \
     --dry-run=client -o yaml > deployment.yaml

kubectl apply -f deployment.yaml
```

## Rolling and Rollback Update (Deployment Strategies)

Deployment Strategies

1. Rolling Update (default)

2. Recreate -> Deletes all old pods first, then creates new ones.

3. Blue-Green -> Two identical environments (blue and green). One is live, the other is idle. When you deploy a new version, you switch traffic to the new environment.

4. Canary -> Gradually roll out the new version to a small subset of users before a full rollout.

5. A/B Testing -> Similar to Canary, but focuses on testing different versions of the application with different user groups.

6. Shadow Deployment -> Runs the new version alongside the old one, but only sends a copy of the traffic to the new version for testing.

A **rollout** is the process of deploying a new version of your application (e.g., changing the container image, or environment variables).

If something goes wrong during a rollout (e.g., bad image, app crash), you can **roll back** to the previous working version.

```bash
kubectl set image deployment <name of deployment> <name of container>=<new image name> # update image

kubectl rollout status deployment <name of deployment> # see status

kubectl rollout history deployment <name of deployment> # see history 

kubectl rollout undo deployment/webapp # rollback to previous version

kubectl rollout undo deployment/webapp --to-revision=2 # rollback to specific version

```



### References

1. https://devopscube.com/kubernetes-deployment-tutorial/
2. https://kubernetes.io/docs/concepts/workloads/controllers/deployment/?ref=devopscube.com#rolling-back-a-deployment