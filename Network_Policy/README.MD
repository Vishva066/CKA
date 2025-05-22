# Network Policy

By default, all pods in a Kubernetes cluster can communicate with each other. Network policies are used to control the communication between pods and restrict access to certain pods or namespaces.

## Egress

Egress rules are used to control the outbound traffic from a pod. By default, all pods can send traffic to any destination. Egress rules can be used to restrict this traffic.

## Ingress

Ingress rules are used to control the inbound traffic to a pod. By default, all pods can receive traffic from any source. Ingress rules can be used to restrict this traffic.

## Network Policy

These network policy are used to control the communication between pods. Network policy is applied to a specific namespace.

You can use network policy if you want to prevent apps from reaching each other over the network, which will help limit the damage if one of your apps is compromised.

Types of target endpoints:

- Specific Pods (Pods matching a label are allowed)
- Specific Namespaces (all Pods in the namespace are allowed)
- IP address blocks (endpoints with an IP address in the block are allowed)

To implement network policies we need a network plugin. We have to install a network plugin. Some of the popular network plugins are 

- Calico
- Flannel
- Cilium

Eg

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: pod2
  policyTypes:
    - Ingress
    - Egress
```

## Selectors

- Pod Selector: Selects pods based on labels 

```yaml
podSelector:
  matchLabels:
    app: demo
```

- namespaceSelector: Selects namespaces based on labels

```yaml
namespaceSelector:
  matchLabels:
    app: demo

namespaceSelector:
  matchLabels:
    kubernetes.io/metadata.name: demo-namespace
```

- ipBlock: Selects IP blocks based on CIDR notation

```yaml
ipBlock:
  cidr: 10.0.0.0/24
```

## Port ranges

Adding a ports field to your Ingress and Egress rules lets you restrict this to just the ports your app actually requires:

```yaml
ingress:
  - from:
      - podSelector:
          matchLabels:
            app: demo
    ports:
      - protocol: TCP
        port: 32000
        endPort: 32100
```

## Deny traffice examples

- Deny all traffic to a Pod

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: network-policy
spec:
  podSelector:
    matchLabels:
      app: demo
  policyTypes:
    - Ingress
    - Egress
```

- Deny all traffic to all pods

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: network-policy
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

- Deny all ingress traffic

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: network-policy
spec:
  podSelector: {}
  policyTypes:
    - Ingress
```

- Deny all egress traffic

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: network-policy
spec:
  podSelector: {}
  policyTypes:
    - Egress
```

- Allow all traffic to a Pod

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: network-policy
spec:
  podSelector:
    matchLabels:
      app: demo
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - {}
  egress:
    - {}

``` 

### Sample Question and Answer

my-app-deployment and cache-deployment deployed, and my-app-deployment deployment exposed through a service named my-app-service . Create a NetworkPolicy named my-app-network-policy to restrict incoming and outgoing traffic to my-app-deployment pods with the following specifications:

Allow incoming traffic only from pods.
Allow incoming traffic from a specific pod with the label app=trusted
Allow outgoing traffic to pods.
Deny all other incoming and outgoing traffic.

Ans:

```yaml
d: NetworkPolicy
metadata:
  name: my-app-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector: {}
  - from:
    - podSelector:
        matchLabels:
          app: trusted
  egress:
  - to:
    - podSelector: {}
```