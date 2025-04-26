# RBAC

RBAC is like setting permissions in Kubernetes:

- Who can do

- What actions

- On Which resources

The main object in RBAC are

- Role

- ClusterRole

- RoleBinding

- ClusterRoleBinding

- Service Account


The RBAC can be created using two ways 

- Imperative way
- Declarative way

## Syntax

### Role Syntax

Imperative way

```bash
kubectl create role <role-name> --verb=<verb> --resource=<resource> --namespace=<namespace>
```

Declarative way

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: <role-name>
  namespace: <namespace>
rules:
- apiGroups: [""] # Indicates the API group(Here it indicates core group that is where pod is defined)
  # The empty string indicates the core API group
  resources: ["<resource>"] # Resources that the role can access
  verbs: ["<verb>"] # Action it can perform
```

### ClusterRole Syntax

Imperative way

```bash
kubectl create clusterrole <cluster-role-name> --verb=<verb> --resource=<resource>
```

Declarative way

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: <name>
rules:
- apiGroups: ["metrics.k8s.io"]
  resources: ["<resource_name>", "<resource_name>"]
  verbs: ["<verb>", "<verb>"]
```

### RoleBinding Syntax

Imperative way

```bash
kubectl create rolebinding <role-binding-name> --role=<role-name> --serviceaccount=<namespace>:<service-account-name> --namespace=<namespace>
```

Declarative way

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: <role-binding-name>
  namespace: <namespace>
subjects:
- kind: ServiceAccount
  name: <service-account-name>
  namespace: <namespace>
roleRef:
  kind: Role
  name: <role-name>
  apiGroup: rbac.authorization.k8s.io # Need to mention the apiGroup here also then only it will know who to contact to
```

### ClusterRoleBinding Syntax

Imperative way

```bash

kubectl create clusterrolebinding <binding-name> --clusterrole=<cluster-role-name> --user=<user-name>
```

Declarative way

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: <binding-name>
subjects:
- kind: <kind>
  name: <name>
roleRef:
    kind: ClusterRole
    name: <cluster-role-name>
    apiGroup: rbac.authorization.k8s.io
```

### Service Account Syntax

Imperative way

```bash
kubectl create serviceaccount <service-account-name> --namespace=<namespace>
```

Declarative way

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: <name>
  namespace: <namespace>
```

## Imperative way

Imperative way creating the resources directly using the command line using **kubectl create** command.

Eg

```bash
kubectl create namespace dev # Create a namespace

kubectl create serviceaccount sa -n dev # Create a service account named sa in dev namespace

kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods # Create a cluster role named pod-reader with get, list, watch verbs on pods resource

kubectl create clusterrolebinding pod-reader-binding --clusterrole=pod-reader --serviceaccount=dev:sa # Create a cluster role binding named pod-reader-binding that binds the pod-reader cluster role to the sa service account in dev namespace

kubectl create role pod-reader --verb=get,list,watch --resource=pods --namespace=dev # Create a role named pod-reader with get, list, watch verbs on pods resource in dev namespace

kubectl create rolebinding pod-reader-binding --role=pod-reader --serviceaccount=dev:sa --namespace=dev # Create a role binding named pod-reader-binding that binds the pod-reader role to the sa service account in dev namespace

```

To check if the particular user/subject has access use this command

```bash

kubectl auth can-i list pods --namespace=dev --as system:serviceaccount:dev:sa

```

- list pods - Action to perform

- namespace=dev - Namespace where the action is going to be performed

- as system:serviceaccount:dev:sa - The user/subject who is going to perform the action

## Declarative way

Declarative way creating the resources using YAML files and applying them using **kubectl apply** command.

Role Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1 # Api version for RBAC
kind: Role
metadata:
  name: pod-reader # Name of the role
  namespace: dev # Namespace where the role is applied to
rules:
- apiGroups: [""] # Indicates the API group(Here it indicates core group that is where pod is defined)
  # The empty string indicates the core API group
  resources: ["pods"] # Resources that the role can access
  verbs: ["get", "watch", "list"] # Action it can perform
```

ClusterRole Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: all-read
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
```

RoleBinding Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding 
metadata:
  name: alice-pod-reader # Name of the rolebinding resource
  namespace: dev # Namespace where the rolebinding is created
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: dev
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

CluserRoleBinding Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: carol-node-reader
subjects:
- kind: User
  name: carol
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```