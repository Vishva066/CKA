# Service

In Kubernetes, a Service is a method for exposing a network application that is running as one or more Pods in your cluster.

The Service API, part of Kubernetes, is an abstraction to help you expose groups of Pods over a network.

The set of Pods targeted by a Service is usually determined by a selector that you define.

## Service Types

- ClusterIP: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster.

- NodePort: Exposes the Service on each Node’s IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created.

- LoadBalancer: Exposes the Service externally using an external load balancer. Kubernetes does not directly offer a load balancing component; you must provide one, or you can integrate your Kubernetes cluster with a cloud provider.

- ExternalName: Maps the Service to the contents of the externalName field (e.g., foo.bar.example.com), by returning a CNAME record with its value. No proxying of any kind is set up.

## ClusterIP

This default Service type assigns an IP address from a pool of IP addresses that your cluster has reserved for that purpose.

You can specify your own cluster IP address as part of a Service creation request. To do this, set the .spec.clusterIP field. 

Here is an example of a ClusterIP Service manifest:

```yaml
apiVersion: v1
kind: Service
metadata:
    name: my-clusterip-service
    namespace: default
spec:
    selector:
        app: my-app
    ports:
        - protocol: TCP
            port: 80
            targetPort: 8080
    clusterIP: 10.96.0.100
```

In this example:
- The `selector` matches Pods with the label `app: my-app`.
- The Service listens on port `80` and forwards traffic to port `8080` on the selected Pods.
- The `clusterIP` is explicitly set to `10.96.0.100`. If omitted, Kubernetes will automatically assign an IP.

## NodePort

If you set the type field to NodePort, the Kubernetes control plane allocates a port from a range specified by --service-node-port-range flag (default: 30000-32767). 

Using a NodePort gives you the freedom to set up your own load balancing solution, to configure environments that are not fully supported by Kubernetes, or even to expose one or more nodes' IP addresses directly.


Here is an example of a NodePort Service manifest:

```yaml
apiVersion: v1
kind: Service
metadata:
    name: my-nodeport-service
    namespace: default
spec:
    type: NodePort
    selector:
        app: my-app
    ports:
        - protocol: TCP
          port: 80
          targetPort: 8080
          nodePort: 31000
```

In this example:
- The `selector` matches Pods with the label `app: my-app`.
- The Service listens on port `80` and forwards traffic to port `8080` on the selected Pods.
- The `nodePort` is explicitly set to `31000`. If omitted, Kubernetes will automatically assign a port within the default range (30000-32767).
- The Service is accessible externally via `<NodeIP>:31000`.

## LoadBalcer

On cloud providers which support external load balancers, setting the type field to LoadBalancer provisions a load balancer for your Service.

Eg 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  clusterIP: 10.0.171.239
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - ip: 192.0.2.127
```

## ExternalName

Services of type ExternalName map a Service to a DNS name, not to a typical selector such as my-service or cassandra. You specify these Services with the spec.externalName parameter.

This Service definition, for example, maps the my-service Service in the prod namespace to my.database.example.com:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

## Headless Service


A Headless Service in Kubernetes is a Service that does not allocate a ClusterIP. Instead, it allows you to directly access the individual Pods that are part of the Service. 

This is useful when you want to manage the load balancing yourself or when you need to discover the individual Pods for stateful applications.

To create a Headless Service, you set the `.spec.clusterIP` field explicitly to `None`.

Here is an example of a Headless Service manifest:

```yaml
apiVersion: v1
kind: Service
metadata:
    name: my-headless-service
    namespace: default
spec:
    clusterIP: None
    selector:
        app: my-app
    ports:
        - protocol: TCP
            port: 80
            targetPort: 8080
```

In this example:
- The `selector` matches Pods with the label `app: my-app`.
- The Service does not allocate a ClusterIP (`clusterIP: None`).
- The Service listens on port `80` and forwards traffic to port `8080` on the selected Pods.
- DNS records are created for the individual Pods, allowing direct access to them.
- This is commonly used in StatefulSets or when you need fine-grained control over Pod communication.

### References

1. https://kubernetes.io/docs/concepts/services-networking/service/