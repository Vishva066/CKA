# Questions

1. List cert-manager CRDs and extract subject field from Certificate CRD

Answer:

```bash
kubectl get crd | grep cert-manager > cert-manager.txt
```

To extract the subject field for Certificate crd execute this command

```bash
kubectl get crd certificates.cert-manager.io -o=jsonpath='{.spec.versions[*].schema.openAPIV3Schema.properties.spec.properties.subject}' > subject.yaml
```

Resources:

1. https://cert-manager.io/v1.1-docs/installation/kubernetes/
2. https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/
3. https://kubernetes.io/docs/reference/kubectl/jsonpath/


2. Edit ConfigMap to enable TLSv1.2 and make it immutable

Answer:

```bash
kubectl create configmap tlsconfig --from-literal=TLS_MIN_VERSION=TLSv1.2 --dry-run=client -o yaml > tlsconfig.yaml
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: tls-config
immutable: true
data:
  TLS_MIN_VERSION: TLSv1.2
```

**Note the immutable should be in the same level as metadata**

3. Install cri-dockerd .deb and set system params.

Prepare your environment for Kubernetes cluster deployment using kubeadm. Adjust and persist the following network parameters:

net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1


Answer

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sudo sysctl --system
```

To verfiy check this out 

```bash
sysctl net.ipv4.ip_forward
sysctl net.bridge.bridge-nf-call-iptables
```

```bash
sudo dpkg -i cri-dockerd_0.3.1.3-0.ubuntu-jammy_amd64.deb

sudo systemctl enable cri-docker.service

sudo systemctl start cri-docker.service
``` 

It will be better if we first set up the env's and then execute these 

If you want to add an variable to k8s.conf use sudo tee -a (apped)

Otherwise it will overwrite the existing

Resources:
1. https://kubernetes.io/docs/setup/production-environment/container-runtimes/
2. https://www.youtube.com/watch?v=b8BYiMawi8A



4. Create Ingress for a Deployment with hostname app.example.com and the path is echo and backend service name is echo-service

Answer

```bash
kubectl create ingress echo-ingress --rule=app.example.com/echo=echo-service:3000 --dry-run=client -o yaml > ingress.yaml

sudo nano ingress.yaml
```

Change the path type from exact to Prefix

```bash
kubectl apply -f ingress.yaml
```

Resources:
1. https://kubernetes.io/docs/concepts/services-networking/ingress/


5. Gateway API Migration (Ingress ➝ Gateway + HTTPRoute)

Answer:

1. Create Gateway
2. Create HTTPRoute
3. Delete Ingress


First check out the gateway Class used 

```bash
kubectl get gatewayclass
```

Create the gateway

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: example-gateway
spec:
    gatewayClassName: example-class
    listeners:
        - name: http
          protocol: HTTP
          port: 80
          hostname: "app.example.com"
          tls:
            mode: Terminate
            certificateRefs:
              - name: secret
                kind: Secret
```

Create the HTTP Route

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: example-httproute
spec:
    parentRefs:
        - name: example-gateway
    hostnames:
    - "app.example.com"
    rules:
        - matches:
            -path:
                type: PathPrefix
                value: /login
        backendRefs:
        - name: example-svc
          port: 8080
```

Delete the ingress

```bash
kubectl delete ingress <ingress-name>
```

6.  Add sidecar and shared volume

Answer

```yaml
volumes:
- name: shared-data
  emptyDir: {}

containers:
- name: main
  image: myapp
  volumeMounts:
  - name: shared-data
    mountPath: /app/shared

- name: sidecar
  image: alpine
  command: ["/bin/sh", "-c", "tail -f /dev/null"]
  volumeMounts:
  - name: shared-data
    mountPath: /sidecar/shared
```

7. PVC + Mount to Deployment

Answer:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: foo-pvc
  namespace: foo
spec:
  volumeName: foo-pv
  accessMode:
    - ReadWriteOnce
  resources:
    requests:
        storage: 1Gi
```

In pVC mainly thrre things to notice

1. accessMode: 
2. volumeName
3. resources.requests.storage

```yaml
volumes:
- name: storage
  persistentVolumeClaim:
    claimName: data-pvc

containers:
- name: app
  image: myapp
  volumeMounts:
  - name: storage
    mountPath: /data
```

8. ArgoCD Install with Helm No CRDs and with CRDs

Answer

- Add the Helm repo using this command

```bash
helm repo add argo https://argoproj.github.io/argo-helm

helm repo update
```

- Create a manifest for installing without crds

```bash
helm template argocd argo/argo-cd \
--namespace argocd \
--version 8.1.2 \
--set crds.install=false > argocd-disabled.yaml

kubectl create namespace argocd

kubectl apply -f argocd-disabled.yaml
```

- Create a manifest for installing with crds

```bash
helm template argocd argo/argo-cd \
--namespace argocd \
--version 8.1.2 \
--set crds.install=true > argocd-enabled.yaml

kubectl apply -f argocd-enabled.yaml
```

Resources:

1. https://artifacthub.io/packages/helm/argo/argo-cd
2. https://www.youtube.com/watch?v=U5DfmuWU58E

9. Divide Node Resources (Include Init Containers)

Answer:

Check the node resources 

```bash
kubectl top node <node_name>

kubectl describe node <node_name>

```

Now based on this allocate the required memory and limits also leave some memory and cpu for overhead

```yaml
initContainers:
- name: init
  image: busybox
  resources:
    requests:
      cpu: "100m"
      memory: "128Mi"
    limits:
      cpu: "100m"
      memory: "128Mi"

containers:
- name: app
  resources:
    requests:
      cpu: "500m"
      memory: "256Mi"
    limits:
      cpu: "500m"
      memory: "256Mi"
```

10. Least Permissive NetworkPolicy

Answer

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-to-backend
  namespace: frontend
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
    - Egress
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              name: back-end
          podSelector:
            matchLabels:
              app: backend
```

For these type of questions always select the one with namespace selector or pod selector don't selct with ip block as it will become overly protective and sometimes break.


11. Install CNI: Flannel vs Calico

Answer:

First before installing the calico CNI first we have to check the cidr assigned to the cluster to check that use this command

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.2/manifests/tigera-operator.yaml

curl https://raw.githubusercontent.com/projectcalico/calico/v3.30.2/manifests/custom-resources.yaml -O



```

Here it will display the cluster-cidr note that and make sure it matches the calico cidr.

```bash
wget <url>

cat <file> | grep -A5 default-ipv4-ippool

kubectl create -f custom-resources.yaml
```

Normally it would be commented. Uncomment it and enter the IPv4 Address

By default the calico cidr is 192.168.0.0/16 

12. HPA - Horizontal Pod Autoscaler

Create a HPA autoscaler with max 4 replicas in case cpu percent of 50% also add the stabilisation window as 30 sec

Answer:

```bash
kubectl autoscale deployment nginx --cpu-percent=50 --min=1 --max=4 --dry-run=client -o yaml > hpa.yaml
```

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  creationTimestamp: null
  name: nginx
spec:
  maxReplicas: 4
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 30
  metrics:
  - resource:
      name: cpu
      target:
        averageUtilization: 50
        type: Utilization
    type: Resource
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
status:
  currentMetrics: null
  desiredReplicas: 0
```

Make sure the behaviour is in the same field as maxReplicas.

13. Expose Deployment via NodePort and Fix Port in Deployment

Answer:

```bash
kubectl expose deployment nginx --port=80 --target-port=80 --name=nginx-https --type=NodePort --dry-run=client -o yaml > node.yaml
```

Now change the target port to the desired port number

14. Priority Class

Answer

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

15. Troubleshooting

Kube api server is not working because it was previously configured with an external etcd cluster. Now make sure it is working

Answer:

Mostly the error will be because ther etcd server port is mismatching

First check the kube api server logs 

```bash
crictl ps -a | grep kube-api
```

Check the logs if it is not running or exited or not found anything check the logs of the pods in the following location.

```bash
cd /var/log/pods/
```

In this check the logs of the kube-api-server and then go to the manifests file and change the etcd-cluster-server ip to 2379.

Before that check the port where etcd is running.

Etcd servers in the kubeapiserver should match the port running in etcd client listen url.(2379)

**After this restart the kubelet pod**

Then the changes will take place.

Also take note whether all the pods are running or not in case if it is not running use describe command to check the logs of the pods.

In my case kube-scheduler was not running because of high cpu requests

16. Create a storage class and make it as default

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-sc
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

Storage class consists of three main things

1. Defauklt or not annotation
2. provisioner
3. volumeBindingMode

17. Gateway API to support HTTPS

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: web-gateway
  namespace: cka5673
spec:
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    hostname: "gateway.web.k8s.local"
    tls:
      mode: Terminate
      certificateRefs:
      - name: web-tls
        kind: Secret
        group: ""
```