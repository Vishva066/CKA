# Argo CD

Argo CD is described as “a declarative, GitOps continuous delivery tool for Kubernetes.” It can monitor your source repositories and automatically deploy changes to your cluster.

Argo CD continuously monitors Git repositories and compares the actual cluster state to the declared configuration. When differences are detected, it can alert users or automatically apply changes to bring the system into the desired state.

## Featurees

- GitOps based deployment

- Declarative application definitions (Helm, Kustomize, Jsonnet, YAML)

- Automated Sync -> Syncs k8s resources with git and matches the cluster state with the desired state

- Real time application status monitoring

- RBAC

- MultiCluster Support

- Web UI and CLI

## Argo CD Components

Core elements of Argo CD architecture

- Application Controller -> This is the component we install in our cluster. 

- Application -> A grpup of K8s resources that collectively deploy workload

- Live state

- Target state -> State declared in git

- Refresh -> A refresh occurs when Argo fetches the target state from your repository. It compares the changes against the live state but doesn’t necessarily apply them at this stage.

- Sync -> A Sync is the process of applying the changes discovered by a Refresh. Each  Sync moves the cluster back toward the target state.

- API server -> The Argo API server provides the REST and gRPC API interface used by the CLI, Web UI, and external integrations.

- Git repo

## Pull Based CI/CD

Argo CD is a pull-based CI/CD system. It runs inside your Kubernetes cluster and pulls source from your repositories. Argo then applies the changes for you without a manually configured pipeline.

This model is more secure than push-based workflows. You don’t have to expose your cluster’s API server or store Kubernetes credentials in your CI/CD platform. Compromising a source repository only gives an attacker access to your code instead of the code and a route to your live deployments.

## EG of Argo CD

https://spacelift.io/blog/argocd

## Installation of Argo CD

1. Install argo CD CLI latest release using this method

TO check out the latest release go to the github releases of argo cd and check the LTS version

```bash
wget https://github.com/argoproj/argo-cd/releases/download/v3.0.1/argocd-linux-amd64
```

2. Make it executable using this command

```bash
sudo chmod +x argocd-linux-amd64

sudo mv argocd-linux-amd64 /usr/local/bin/argocd
```

3. To verify that it has been installed execute this command

```bash
argocd version
``` 

4. Now install argocd in kubernetes cluster using this method

```bash
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v3.0.1/manifests/install.yaml
```

5. Connect to the argocd pod using UI by port-forwarding method

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Get the password using this method

```bash
argocd admin initial-password -n argocd
```

6. Login using CLI using this method

```bash
argocd login localhost:8080
```

7. Deploy ArgoCD application using this command

```bash
argocd app create argo-demo \
  --repo https://github.com/<username>/<repo>.git \
  --path . \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

-  --repo flag specifies the URL of your Git repository.

- --path flag instructs Argo to search for Kubernetes manifests, Helm charts, and other deployable assets inside this path within your repo. . is used here because the example manifests are stored in the repo’s root.

- --dest-server flag specifies the URL of the Kubernetes cluster to deploy to. You can use kubernetes.default.svc when you’re deploying to the same cluster that Argo’s running in.

- --dest-namespace sets the Kubernetes namespace that your app will be deployed into. This should match the metadata.namespace fields set on your resources.

8. To check whether it is created or not execute this command

```bash
argocd apps list
```

9. To sync the application with github execute this command

```bash
argocd app sync argo-demo
```
