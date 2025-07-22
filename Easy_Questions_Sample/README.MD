# Simple Questions

1. Scale the deployment named dragon to 8 replicas (Scaling)


```bash
kubectl scale deployment/dragon --replicas=8
```

2. 
    1. Edit the deployment named deppy to expose TCP port 80 and name the port http.
    2. Create a nodeport Svc named deppysvc to expose the pods in the deployment
    (ClusterIP, NodePort, LB)

```bash

KUBE_EDITOR="nano"

kubectl edit deployment/mydeployment

```

```yaml
ports:
    - containerPort: 80
      name: http
      protocol: TCP
```

```bash
kubectl create service nodeport deppsvc --tcp=80:80 --dry-run=client -o yaml > nodeport.yaml

kubectl expose deployment deppy --type=NodePort
```

Syntax

kubectl create service nodeport <svc-name> --tcp=port:targetport

- port: 80: Other resources in the cluster will send traffic to the Service on port 80.
- targetPort: 8080: The Service will forward that traffic to the Pods on port 8080.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: deppysvc
spec:
  type: NodePort
  selector:
    app: deppy
  ports:
    - port: 80
      # By default and for convenience, the `targetPort` is set to
      # the same value as the `port` field.
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane
      # will allocate a port from a range (default: 30000-32767)
      nodePort: 30007
```


3. 
    1. Create a PVC named rwopvc that does the following
    2. Its capacity is 4Gi
    3. Access mode of RWO
    4. Mount to a pod named rwopod at the mount path /var/www/html

First lets create a PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: rwopv
spec:
  accessMode:
    - ReadWriteOnce
  capacity:
    storage: 4Gi
  storageClassName: standard
  hostPath:
    path: /rwo
```

Create a pvc

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rwopvc
spec:
  requests:
    storage: 4Gi
  accessMode:
    - ReadWriteOnce
  volumeName: rwo
```

Create a pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: rwopod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: rwopvc
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/var/www/html"
          name: task-pv-storage
```

4. Create a Persistent Volume named rompv with the access mode ReadOnlyMany and a capacity of 6Gi

Answer

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: rompv
spec:
  accessMode:
    - ReadOnlyMany
  capacity:
    storage: 6Gi
  hostPath:
    path: /var/rom
```

5. Create an ingress resource named luau that routes traffic on the path /aloha to the aloha servie on port 54321

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: luau
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /aloha
        pathType: Prefix
        backend:
          service:
            name: aloha
            port:
              number: 54321
```

6. 
  1. Identify all the pods in the integration namespace with the label app=intensivve
  2. Determine which of these Pods is using the most CPU resources.
  3. Write the name of the Pod consuming the most CPU resources to /opt/cka/answers/cpu_pod_01.txt

Answer

```bash
kubect get pod -l app=intensive -n integration


kubectl top pod -l app=intensive --namespace=integration > /opt/cka/answers/cpu_pod_01.txt
```

7. Create a Pod named noded that uses the nginx image.
  Ensure the Pod is scheduled to a node labeled disk=nvme

Answer

```bash
kubectl run noded --image=nginx --dry-run=client > nginx.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: noded
  name: noded
spec:
  nodeSelector:
    disk: nvme
  containers:
  - image: nginx
    name: noded
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

(OR)

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: noded
  name: noded
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disk
            operator: In
            values:
            - nvme
  containers:
  - image: nginx
    name: noded
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```


8. Create a Pod named multicontainer that has two containers

  1. A container running the redis:6.26 image
  2. A container running the nginx:1.21.6 image

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: noded
  name: noded
spec:
  containers:
  - image: nginx:1.21.6
    name: nginx
    resources: {}
  - image: redis:6.2.6
    name: redis
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

9. Monitor the log of the fnf pod and filter any lines containing the error file not found. /opt/cka/answers/soreted_log.log

```bash
kubectl logs fnf-pod | grep "file-not-found" > /opt/cka/answers/soreted_log.log
```

10. Add a sidecar container using the busybox image to the existing Pod logger. 
  The container should be mounted at the path /var/log and run the command /bin/sh -c tail -f /var/log/log01.log

```yaml
containers:
  - name: sidecar
    image: alpine:latest
    restartPolicy: Always
    command: ['/bin/sh', '-c', 'tail -f /var/log/log01.log']
    volumeMounts:
      - name: data
        mountPath: /var/log
```

11. Create a cluster role named app-creator that allows create permission for Deployments, Statefulsets and Daemonsets.
  Create a service account named app-dev.
  Bind the service account app-dev to the ClusterRole app-creator using a ClusterRoleBinding

```bash
kubectl create clusterrole app-creator --verb=create --resource=deployment,statefulset,daemonset

kubectl create serviceaccount app-dev

kubectl create clusterrolebinding --role=app-create --serviceaccount=default:app-dev
```

12. Inspect the nodes for any taints they have. Write the results to the files

```bash
kubectl describe node1 | grep Taint > node1.txt

kubectl describe node2 | grep Taint > node2.txt
```

13. Create a configmap called metal-cm containing the file ~/mycode/yaml/metal.html
  To the deployment "enter-sandman" add the metal-cm configmap mounted to the path /var/www/index.html

```bash
  kubectl create configmap metal-cm --from-file=~/mycode/yaml/metal.html
```

```bash
kubectl get deployment enter-sandman --dry-run=client -o yaml > metalcm.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: registry.k8s.io/busybox:1.27.2
      command: [ "/bin/sh", "-c", "ls /etc/config/" ]
      volumeMounts:
      - name: config-volume
        mountPath: /var/www/index.html
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want
        # to add to the container
        name: metal-cm
  restartPolicy: Never
```


14. Create a new secret named juicy secret it must contain the following values.
  username=kiwis, password=aredelicious

Edit the pod so it is available from the following as env USERKIWI and PASSKIWI

```bash
kubectl create secret generic my-secret --from-literal=username=kiwis --from-literal=password=aredelicious
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    env:
      - name: USERKIWI
        valueFrom:
          secretKeyRef:
            name: my-secret
            key: username
      - name: PASSKIWI
        valueFrom:
          secretKeyRef:
            name: my-secret
            key: password
``` 

15. In ns cherry there are two deployments named pit and stem. Both deployments are exposed via service. 

Make a Network Policy named cherry-control that prevents outgoing traffic from deployment pit except to that of deployment stem.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cherry-control
  namespace: cherry
spec:
  podSelector:
    matchLabels:
      role: pit
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          role: stem
    ports:
    - protocol: TCP
      port: 5978


```

16. Modify the helm chart configuration to ensure 3 replicas are created. Then install the chart into the cluster 


Answer:

Change the values.yaml file replicas count to 3

17. There is an existing deployment named mufasa in ns king of lions
  Check the deployment history and rollback to a previous version


```bash
kubectl rollout history deployment mufasa -n king of lions

kubectl rollout undo deployment/mufasa -n king og lions --to-revision=2
```

18. Join node 2 to your existing kubeadm cluster. It has already been provisioned.

```bash
kubeadm join <your-cluster-endpoint> --token <your-token> --discovery-token-ca-cert-hash sha256:<your-hash>
```

19. Outline the Helm steps to do the following

- Add and update the argo helm repo

- Install argocd v8.0.17 with CRD enabled in argocd

- Install argocd v8.0.17 with CRDs disabled in argo-no-crds

- Render both release to argo-cd-crds-enabled.yaml and argo-cd-crds-disabled.yaml

Answers

```bash

helm repo add <argocd-url-helm>

helm repo update

helm search repo argo/argo-cd --versions
```

```bash
helm show values argo/argo-cd --version 8.0.17 | grep crds -E5
```

```bash
helm template argocd argo/argo-cd --namespace argo-cd --set crds.install=false > argo-cd-crds-disabled.yaml

helm template argocd argo/argo-cd --namespace argo-cd > argo-cd-crds-enabled.yaml
```