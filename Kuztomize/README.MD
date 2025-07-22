# Kuztomize

Kustomize is an open-source configuration management tool designed for Kubernetes. 

It empowers developers and operators to customize, **patch**, and manage Kubernetes manifests in a declarative manner.

Kustomize operates by allowing users to overlay their base Kubernetes manifests with customizations. This is achieved through the use of “kustomization.yaml” files, where users can specify patches, variables, and other configuration details.

**The true power of Kustomize lies in its ability to apply patches and overlays to base manifests.**

## Kustomize Tutorial

1. Install Kustomize using the following command

```bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
```

In some directory containing your YAML [resource]
files (deployments, services, configmaps, etc.), create a
[kustomization] file.

This file should declare those resources, and any
customization to apply to them, e.g. _add a common
label_.

```

base: kustomization + resources

kustomization.yaml                                      deployment.yaml                                                 service.yaml
+---------------------------------------------+         +-------------------------------------------------------+       +-----------------------------------+
| apiVersion: kustomize.config.k8s.io/v1beta1 |         | apiVersion: apps/v1                                   |       | apiVersion: v1                    |
| kind: Kustomization                         |         | kind: Deployment                                      |       | kind: Service                     |
| labels:                                     |         | metadata:                                             |       | metadata:                         |
| - includeSelectors: true                    |         |   name: myapp                                         |       |   name: myapp                     |
|   pairs:                                    |         | spec:                                                 |       | spec:                             |
|     app: myapp                              |         |   selector:                                           |       |   selector:                       |
| resources:                                  |         |     matchLabels:                                      |       |     app: myapp                    |
|   - deployment.yaml                         |         |       app: myapp                                      |       |   ports:                          |
|   - service.yaml                            |         |   template:                                           |       |     - port: 6060                  |
| configMapGenerator:                         |         |     metadata:                                         |       |       targetPort: 6060            |
|   - name: myapp-map                         |         |       labels:                                         |       +-----------------------------------+
|     literals:                               |         |         app: myapp                                    |
|       - KEY=value                           |         |     spec:                                             |
+---------------------------------------------+         |       containers:                                     |
                                                        |         - name: myapp                                 |
                                                        |           image: myapp                                |
                                                        |           resources:                                  |
                                                        |             limits:                                   |
                                                        |               memory: "128Mi"                         |
                                                        |               cpu: "500m"                             |
                                                        |           ports:                                      |
                                                        |             - containerPort: 6060                     |
                                                        +-------------------------------------------------------+

```

File structure:

> ```
> ~/someApp
> ├── deployment.yaml
> ├── kustomization.yaml
> └── service.yaml
> ```

The resources in this directory could be a fork of
someone else's configuration.  If so, you can easily
rebase from the source material to capture
improvements, because you don't modify the resources
directly.

Generate customized YAML with:

```
kustomize build ~/someApp
```

The YAML can be directly [applied] to a cluster:

> ```
> kustomize build ~/someApp | kubectl apply -f -
> ```


### 2) Create [variants] using [overlays]

Manage traditional [variants] of a configuration - like
_development_, _staging_ and _production_ - using
[overlays] that modify a common [base].

```

overlay: kustomization + patches

kustomization.yaml                                      replica_count.yaml                      cpu_count.yaml
+-----------------------------------------------+       +-------------------------------+       +------------------------------------------+
| apiVersion: kustomize.config.k8s.io/v1beta1   |       | apiVersion: apps/v1           |       | apiVersion: apps/v1                      |
| kind: Kustomization                           |       | kind: Deployment              |       | kind: Deployment                         |
| labels:                                       |       | metadata:                     |       | metadata:                                |
|  - includeSelectors: true                     |       |   name: myapp                 |       |   name: myapp                            |
|    pairs:                                     |       | spec:                         |       | spec:                                    |
|      variant: prod                            |       |   replicas: 80                |       |  template:                               |
| resources:                                    |       +-------------------------------+       |     spec:                                |
|   - ../../base                                |                                               |       containers:                        |
| patches:                                      |                                               |         - name: myapp                    |
|   - path: replica_count.yaml                  |                                               |           resources:                     |
|   - path: cpu_count.yaml                      |                                               |             limits:                      |
+-----------------------------------------------+                                               |               memory: "128Mi"            |
                                                                                                |               cpu: "7000m"               |
                                                                                                +------------------------------------------+
```


File structure:
> ```
> ~/someApp
> ├── base
> │   ├── deployment.yaml
> │   ├── kustomization.yaml
> │   └── service.yaml
> └── overlays
>     ├── development
>     │   ├── cpu_count.yaml
>     │   ├── kustomization.yaml
>     │   └── replica_count.yaml
>     └── production
>         ├── cpu_count.yaml
>         ├── kustomization.yaml
>         └── replica_count.yaml
> ```

Take the work from step (1) above, move it into a
`someApp` subdirectory called `base`, then
place overlays in a sibling directory.

An overlay is just another kustomization, referring to
the base, and referring to patches to apply to that
base.

This arrangement makes it easy to manage your
configuration with `git`.  The base could have files
from an upstream repository managed by someone else.
The overlays could be in a repository you own.
Arranging the repo clones as siblings on disk avoids
the need for git submodules (though that works fine, if
you are a submodule fan).

Generate YAML with

```sh
kustomize build ~/someApp/overlays/production
```

The YAML can be directly [applied] to a cluster:

> ```sh
> kustomize build ~/someApp/overlays/production | kubectl apply -f -
> ```
