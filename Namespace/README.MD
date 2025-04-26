# Namespace

Create a namespace using this command

```bash

kubectl create namespace <namespace-name>
```

Create a namespace declartaive way using this command

```bash
apiVersion: v1
kind: Namespace
metadata:
  name: deployment-demo
  labels:
    apps: web-based
  annotations:
    type: demo
```

Setup resource quoto for the namespace

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-quota
  namespace: deployment-demo
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
```

```bash
kubectl apply -f resource-quota.yaml
```