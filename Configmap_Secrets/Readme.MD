# ConfigMap

A ConfigMap is an API object used to store non-confidential data in key-value pairs.

A ConfigMap has data and binaryData fields.

Eg of config map

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5    
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true    
```

Pod mounting a configmap

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    configMap:
      name: myconfigmap
```
ConfigMap as Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    env:
    - name: PLAYER_INITIAL_LIVES
      valueFrom:
        configMapKeyRef:
          name: game-demo
          key: player_initial_lives
    - name: UI_PROPERTIES_FILE_NAME
      valueFrom:
        configMapKeyRef:
          name: game-demo
          key: ui_properties_file_name
```

# Secrets

A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key.

Secrets are similar to ConfigMaps but are specifically intended to hold confidential data.

***Note**

**Kubernetes Secrets are, by default, stored unencrypted in the API server's underlying data store (etcd).**

**In order to safely use Secrets, take at least the following steps:**

**1. Enable Encryption at Rest for Secrets.**

**2. Enable or configure RBAC rules with least-privilege access to Secrets.**

**3. Restrict Secret access to specific containers.**

**4. Consider using external Secret store providers.**

Secrets use cases

- Set environment variables for a container.

- Provide credentials such as SSH keys or passwords to Pods.

- Allow the kubelet to pull container images from private registries.

- dotfiles in a secret volume -> You can make your data "hidden" by defining a key that begins with a dot. This key represents a dotfile or "hidden" file.

Types of Secrets

- Opaque -> arbitrary user-defined data

    - Opaque is the default Secret type if you don't explicitly specify a type in a Secret manifest. 

- kubernetes.io/service-account-token	-> ServiceAccount token

- kubernetes.io/dockercfg ->	serialized ~/.dockercfg file

- kubernetes.io/dockerconfigjson ->	serialized ~/.docker/config.json file

- kubernetes.io/basic-auth	->credentials for basic authentication

- kubernetes.io/ssh-auth ->	credentials for SSH authentication

- kubernetes.io/tls	-> data for a TLS client or server

- bootstrap.kubernetes.io/token ->	bootstrap token data

## Secrets Format

The values for all keys in the data field have to be base64-encoded strings. 

If the conversion to base64 string is not desirable, you can choose to specify the stringData field instead, which accepts arbitrary strings as values.

**Individual Secrets are limited to 1MiB in size.**

When you reference a Secret in a Pod, you can mark the Secret as optional

If an optional Secret doesn't exist, Kubernetes ignores it.

By default, Secrets are required. None of a Pod's containers will start until all non-optional Secrets are available.

If you want to access data from a Secret in a Pod, one way to do that is to have Kubernetes make the value of that Secret be available as a file inside the filesystem of one or more of the Pod's containers.

```bash
kubectl create secrets <name of secret> --from-file=hello.txt 

kubectl create secrets <name of secret> --from-literal=key1=value1 

```

Declarative way

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  namespace: default
type: Opaque
data:
  username: YWRtaW4=         # base64 encoded "admin"
  password: MWYyZDFlMmU2N2Rm  # base64 encoded "1f2d1e2e67df"
```

## TO decode a secret follow this method

First print out the base64 secret using this method

```bash
kubectl get secret database-data -n database-ns  -o yaml
```

Now decode it using echo command

```bash
echo "c2VjcmV0" | base64 --decode
```
To do it in a single line use this method

```bash
kubectl get secret <secret-name> -n <namespace> -o jsonpath="{.data.<key>}" | base64 --decode
```


## Sample Questions for Configmap and Secrets

1. Create a ConfigMap imperatively
Task:

Create a ConfigMap called app-config with:

Key APP_ENV=production

Key APP_VERSION=1.2

```bash
kubectl create configmap app-config --from-literal=APP_ENV=production --from-literal=APP_VERSION=1.2
```

2. Create a ConfigMap from a file

Create a ConfigMap called file-config from a file /tmp/settings.conf.

```bash
kubectl create configmap file-config --from-file=/tmp/settings.conf

```

3.  Create a Secret imperatively
Task:

Create a Secret called db-secret with:

username = admin

password = password123

```bash
kubectl create secret generic db-secret --from-literal=username=admin --from-literal=password=password123
```

4. Mount ConfigMap into a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
spec:
  containers:
  - name: mycontainer
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

5.  Inject Secret as Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: mycontainer
    image: nginx
    envFrom:
    - secretRef:
        name: db-secret
```

### References

1. https://kubernetes.io/docs/concepts/configuration/configmap/

2. https://kubernetes.io/docs/concepts/configuration/secret/