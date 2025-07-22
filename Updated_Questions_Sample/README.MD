# Questions

1. Create a PVC with 

    a. 100Gi Memory
    
    b. access mode as RWO

    c. using the storage class "local-path"

    d. Mount this PVC at /data

Answer:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
    name: local-path
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: pvc-data
spec:
    accessModes:
        - ReadWriteOnce
    resources:
        requests:
            storage: 100Gi
    storageClassName: local-path
```

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: pod-using-pvc
spec:
    containers:
        - name: app-container
            image: nginx
            volumeMounts:
                - mountPath: /data
                    name: pvc-volume
    volumes:
        - name: pvc-volume
            persistentVolumeClaim:
                claimName: pvc-data
```

2. Create a storage class 

    a. named fast-storage

    b. uses the rancher.io/local-path provisioner

    c. sets retain as the reclaim policy

    d. uses Immediate binding (default)

Create a PV and PVC

    a. The PV should use fast-storage

    b. configure node-affinity so K8s knows where to create the volume

    c. PVC should bind to that PV

    4. Verify when PVC is deleted, PV remains in released state


Answer

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
    name: fast-storage
provisioner: rancher.io/local-path
reclaimPolicy: Retain
volumeBindingMode: Immediatate
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim1
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-storage
  resources:
    requests:
      storage: 30Gi
```

3. Manually create a PV that

    a. Name is static-pv-example

    b. Requests 200Mi

    c. uses hostPath on node-1

    d. access mode ReadWriteOnce

    e. Retain reclaim policy

    f. Create a PVC to bind it.

Answer

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: static-pv-example
spec:
    capacity:
        storage: 200Mi
    accessModes:
        - ReadWriteOnce
    persistentVolumeReclaimPolicy: Retain
    nodeAffinity:
        required:
            nodeSelectorTerms:
            - matchExpressions:
                - key: kubernetes.io/hostname
                  value: node1
    hostPath:
        path: /mnt/node1
        type: DirectoryOrCreate
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: foo-pvc
spec:
  resources:
    requests:
      storage: 200Mi
  volumeName: static-pv-example
```

4. Deploy a sample workload and configure HPA for it. 

    a. Use the deployment `cpu-demo`

    b. Configure an HPA to scale the deployment from 1 up 5 when the CPU utilization is 50%

Answer


```bash
kubectl autoscale deployment cpu-demo --min=1 --max=5 --cpu-percent=50
```

5. Only schedule a pod on that node which has `disktype=ssd` label

Answer

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
    - matchExpressions:
        - key: disktype
        operator: In
        values:
        - ssd         
  containers:
  - name: nginx
    image: nginx:1.14.2
```

6. Enfore the Restricted Pod Security Satandard on the ns restricted-ns.

Pods in their namespace cannot:

    a. Have privileged access
    b. host networking
    c. Have any elevated rights

Answer:

Here we have to enforce pod security with namespace labels

https://kubernetes.io/docs/tutorials/security/ns-level-pss/

https://kubernetes.io/docs/tasks/configure-pod-container/enforce-standards-namespace-labels/

```bash
kubectl label namespae restricted-ns pod-security.kubernetes.io/enforce=restricted

# This enables restricted mode
```

```bash
kubectl label namespace restricted-ns pod-security.kubernetes.io/enforce-version=v1.33
```

7. Node-1 is having a taint. Create a toleration to that pod so that particular pod can be scheduled in node-1 itself

Answer

Taint
```bash
kubectl taint nodes node1 dedicated=experimental:NoSchedule
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeSelector:
    dedicated: experimental
  tolerations:
    - key: "dedicated"
    operator: "Equal"
    value: "experimental"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80

```

8. Deploy a statefulset named web with 2 replicas using the nginx image. Each pod should have its own 1Gi PV for /usr/share/nginx/html. 

Create Headless Service named web for the stateful set.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 2 # by default is 1
  minReadySeconds: 10 # by default is 0
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.24
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "local-path"
      resources:
        requests:
          storage: 1Gi
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
```

9. Fix Core DNS TroubleShooting

Answer

Check the logs of the pods

If you dont get anything describe the pod

10. Use Helm to install Traefik Ingress Controller on the cluster. Create it in a seperate ns also change the helm values to include support for Gateway API.

Answer

```bash
helm repo add traefik https://traefik.github.io/charts

helm repo update
```

```bash
helm show values traefik/traefik | grep kubernetesgateway
```

```bash
helm install traefik traefik/traefik --namespace traefik --create-namespace --set experimental.kubernetesGateway.enabled=true
```

11. Use gateway api for ingress traffic. 

A service named web-service is running in the default namespace on port 80. 

A GatewayAPi compatible  controller is already installed and Gateway class called example-gw-class is available in the cluster.

Tasks:

Use Gateway API resouces to expose the web-service externally on port 80, routed via the hostname web.example.com.

Answer:



12. In the ns limit-test, enforce default resource limits and requests for a container. (POD ADMISSION)

- If a container has no CPU/memory request/limits, assign a default request of 100m and 50Mi CPU and default limit of 200m and 100Mi CPU.

- Prevent any container in this ns from requesting more than 500m memory

Answer:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: resource-constraint
spec:
  limits:
  - default: # this section defines default limits
      cpu: 200m
      memory: 10Mi
    defaultRequest: # this section defines default requests
      cpu: 100m
      memory: 50Mi
    max: # max and min define the limit range
      cpu: 500m
    type: Container
```

13. Migrate an existing application from ingress to Gateway API

- It must follow HTTPS

- Create a gateway named web-gateway with hostname gateway.web.k8s.local that maintains the existing tls and listener configuration from the existing ingress resource named web.

- Create a http route named web-route with hostname gateway.web.k8s.local that maintains the existing routing rules from the current ingress resource named web

Answer

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: web-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    hostname: gateway.web.k8s.local
    tls:
      mode: Terminate
      certificateRefs:
        - kind: Secret
          name: web-tls
```

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
spec:
  parentRefs:
  - name: web-gateway
  hostnames:
  - "gateway.web.k8s.local"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: web-service
      port: 80
```