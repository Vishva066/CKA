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

2. The “kustomization.yaml” file is the main file that is used by the Kustomize tool. In this file, we provide details of our required customizations. The customizations include:

What are the resources that we are going to edit using Kustomize
What are the patches that we are going to apply
Given below is a simple implementation of Kustomize. Consider we have a deployment file with us.

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: api-deployment
  namespace: api-dev

spec:
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
      - name: api
        image: neron/nginx-dev:1.0
```

Now we need to change the namespace and image for this we can create a kustomization.yaml file a provide **required transformers or patches**.

Eg file

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1

kind: Kustomization

# resources section contains the resources that kustomize is handling
resources:
  - deployment.yaml

namespace: api-prod

images:
  - name: neron/nginx-dev:1.0
    newName: neron/nginx-prod
    newTag: "1.0" 
```

**Note: In this case, both the deployment.yaml file and kustomization.yaml file must be on the same directory.**

```bash
kustomize build base-directory/
```

Base directory is where the deployment.yaml and kustomization.yaml file is present

To apply to the k8s cluster follow this command

```bash
kustomize build base-directory/ | kubectl apply -f -

OR

kubectl apply -k base-directory/
```

## Transformers

Transformers are used to create new resources under manifest by applying a series of transformations to the original set of resources.

- commonLabel - Add labels to existing manifest files

- namePrefix of nameSuffix - Add prefix or suffix to existing resource names, we can use  this transformer

- namespace - Add or update namespace from manifests

- images - Add or update images from manifests

## Patches

In Kustomize, we can apply patches to manifest file instead of using “patches” function.

- target -> Where we are going to apply the patch

- operation type -> Operation to perform (add/remove/replace)

- path -> Resource we are going to apply the patch

- value -> With what values we are going to apply

```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace 
        path: /spec/template/spec/containers/0
        value: 
          name: api
          image: nginx
```

