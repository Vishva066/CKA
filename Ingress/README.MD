# Ingress

Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

You must have an Ingress controller to satisfy an Ingress. Only creating an Ingress resource has no effect.

# Ingress Resource

An Ingress needs apiVersion, kind, metadata and spec fields. The name of an Ingress object must be a valid DNS subdomain name.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-resource-backend
spec:
  defaultBackend:
    resource:
      apiGroup: k8s.example.com
      kind: StorageBucket
      name: static-assets
  rules:
    - http:
        paths:
          - path: /icons
            pathType: ImplementationSpecific
            backend:
              resource:
                apiGroup: k8s.example.com
                kind: StorageBucket
                name: icon-assets
```

## Ingress Rules

- An optional host: In this example, no host is specified, so the rule applies to all inbound HTTP traffic through the IP address specified. If a host is provided (for example, foo.bar.com), the rules apply to that host.

- A list of paths (for example, /testpath), each of which has an associated backend defined with a service.name and a service.port.name or service.port.number. Both the host and path must match the content of an incoming request before the load balancer directs traffic to the referenced Service.

- A backend is a combination of Service and port names as described in the Service doc or a custom resource backend by way of a CRD. HTTP (and HTTPS) requests to the Ingress that match the host and path of the rule are sent to the listed backend.

## Ingress Class
Ingress Class is a way to specify which Ingress controller should implement an Ingress resource. It is defined using the `ingressClassName` field in the Ingress resource. This allows multiple Ingress controllers to coexist in a cluster, each handling different Ingress resources based on their class.

If the `ingressClassName` field is not specified, the Ingress resource will be handled by the default Ingress controller, if one is configured.

### Example of Ingress Class

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: example-ingress
spec:
    ingressClassName: nginx
    rules:
        - host: example.com
            http:
                paths:
                    - path: /
                        pathType: Prefix
                        backend:
                            service:
                                name: example-service
                                port:
                                    number: 80
```

In this example:
- The `ingressClassName: nginx` specifies that the Ingress resource should be handled by the Ingress controller associated with the "nginx" Ingress class.
- The rule directs traffic for `example.com` to the `example-service` on port 80.


### References

1. https://devopscube.com/kubernetes-ingress-tutorial/

2. https://devopscube.com/configure-ingress-tls-kubernetes/


