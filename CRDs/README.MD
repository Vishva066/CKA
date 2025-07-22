# Custom Resource

## What is a Custom Resource in Kubernetes?

A **Custom Resource (CR)** in Kubernetes is an extension of the Kubernetes API that allows you to introduce and manage your own resource types, beyond the built-in ones like Pods, Services, or Deployments. Custom Resources enable you to store and retrieve structured data that is specific to your application or domain, making Kubernetes more extensible and adaptable to various use cases.

### Why use Custom Resources?
- To define and manage application-specific configurations declaratively.
- To extend Kubernetes capabilities without modifying its core code.
- To enable automation and integration with Kubernetes-native tooling.

## How to Create a Custom Resource

1. **Define a CustomResourceDefinition (CRD):**
    - A CRD is a Kubernetes resource that tells the API server about your new resource type.
    - You specify the name, schema, and other properties of your custom resource.

2. **Apply the CRD to the cluster:**
    - Use `kubectl apply -f <crd-file>.yaml` to register your new resource type.

3. **Create instances of your Custom Resource:**
    - Once the CRD is registered, you can create objects of your custom type using YAML manifests.

### Example

#### 1. Define a CRD (e.g., for a resource called `CronTab`):

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
    name: crontabs.stable.example.com
spec:
    group: stable.example.com
    versions:
        - name: v1
            served: true
            storage: true
            schema:
                openAPIV3Schema:
                    type: object
                    properties:
                        spec:
                            type: object
                            properties:
                                cronSpec:
                                    type: string
                                image:
                                    type: string
                                replicas:
                                    type: integer
    scope: Namespaced
    names:
        plural: crontabs
        singular: crontab
        kind: CronTab
        shortNames:
            - ct
```

