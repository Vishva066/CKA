# Pod Scheduling

Kubernetes scheduling is the process of selecting a suitable node to run pods.

Kube-scheduler is the default scheduler in K8s, but administrators can use custom schedulers, too.

The most basic approach to scheduling is through the **nodeSelector** available in Kubernetes since version 1.0.

You can specify the nodeSelector in the PodSpec using a key-value pair. If the key-value pair matches exactly the label defined in the node, the pod will get matched to the specific node.

## Node Affinity

The affinity and anti-affinity features in Kubernetes provide administrators with more granular scheduling functionality.

With affinity and anti-affinity, administrators can:

- Define rules, including conditions with logical operators.

- Create “preferred” and “required” rules for a greater variety of matching conditions.

- Match labels of pods running within nodes and determine the scheduling location of new pods.

There are two types of affinity:

- Node affinity: This is similar to nodeSelector but allows for more complex rules and operators.

- Pod affinity: This allows you to specify rules about which pods should be co-located on the same node.

Both node affinity and pod affinity can set required and preferred rules that act as hard and soft rules. You can configure these rules with the following fields in a pod manifest:

- requiredDuringSchedulingIgnoredDuringExecution

- preferredDuringSchedulingIgnoredDuringExecution

Eg

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: name
            operator: In
            values:
            - app-worker-node
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: type
            operator: In
            values:
            - app-01
```

```yaml
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: name
            operator: In
            values:
            - web-app
        topologyKey: topology.kubernetes.io/zone
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 50
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: type
              operator: In
              values:
              - primary
          topologyKey: topology.kubernetes.io/zone
```

## Pod Anti-Affinity

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-test
  labels:
    env: test-env
spec:
  containers:
  - name: nginx
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app-name
            operator: In
            values:
            - database
        topologyKey: topology.kubernetes.io/zone
```

In Kubernetes, the topologyKey is used in affinity and anti-affinity rules to define the scope of the rule—where Kubernetes should look when determining if a pod can be scheduled based on the presence or absence of other pods.

| **topologyKey**                | **Scope of Rule**         | **Meaning**                                                                 |
|--------------------------------|---------------------------|-----------------------------------------------------------------------------|
| `kubernetes.io/hostname`       | Node-level                | Pods must/must not be placed on the same node.                              |
| `topology.kubernetes.io/zone`  | Zone-level (cloud-specific)| Applies across nodes in the same zone.                                      |
| `topology.kubernetes.io/region`| Region-level              | Applies across zones in the same region.                                    |
| Custom label (e.g., `rack`)    | Custom topology           | You can define your own key on nodes and use it for placement rules.        |


## Taints and Tolerations

Taints are set on nodes to repel certain pods.

Tolerations are set on pods to allow them to be scheduled on those tainted nodes.

Think of taints like "Keep Out" signs on a node, and tolerations like permission slips pods can carry to be allowed in.

Eg:

```bash
kubectl taint nodes node1 team=infra:NoSchedule
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  tolerations:
  - key: "team"
    operator: "Equal"
    value: "infra"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx

```

## Sample Questions

Question 1: 

Node Affinity - RequiredDuringScheduling
Create a Pod named web-pod that runs nginx:alpine and should only be scheduled on nodes labeled with tier=frontend.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
   - name: nginx
     image: nginx:alpine
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: tier
            operator: In
            values:
            - frontend
```

Question 2: Node Affinity - PreferredDuringScheduling

Create a Deployment named api-deploy (3 replicas) that prefers nodes with env=prod but will still schedule elsewhere if unavailable.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      lables:
        app: api
    spec:
      containers: 
      - name: app
        image: nginx
      nodeAffinity:
        preferedDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference: 
            matchExpression:
            - key: env
              operator: In
              values:
              - prod
```

Question 3: Taints and Tolerations

A node is tainted with: kubectl taint nodes node1 key=value:NoSchedule. Create a Pod named taint-pod that tolerates this taint and runs nginx.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: taint-pod
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"

```

Question 4: Pod Anti-Affinity

Schedule a Pod named db-pod that should not be co-located with any other Pods labeled app=db on the same node.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: db-pod
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpression:
          - key: app
            operator: In
            values:
            - db
        topologyKey: "kubernetes.io/hostname"
```

Question 5:

Create a Pod that only schedules on nodes labeled zone=east and tolerates a taint with key=dedicated, value=frontend, effect=NoSchedule.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend-pod
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    zone: east
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "frontend"
    effect: "NoSchedule"
```

Question 6:

Create a Deployment of 4 replicas named web-deploy using image nginx. Use preferredDuringScheduling to prioritize nodes labeled tier=web.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: web-deploy
  name: web-deploy
spec:
  replicas: 4
  selector:
    matchLabels:
      app: web-deploy
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web-deploy
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
      affinity:
        nodeAffinity:
          preferedDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpression:
              - key: tier
                values:
                - web
status: {}
```

Question 7:

You have a node labeled app=cache. Create a Pod named memcached that must not be scheduled on nodes with that label.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: memcached
  name: memcached
spec:
  containers:
  - image: nginx
    name: memcached
    resources: {}
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpression:
          - key: app
            operator: In
            values:
            - cache
        topologyKey: "kubernetes.io/hostname"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash

```