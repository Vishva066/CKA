# Init Container

Before the container starts, the init container will run before the main container. The init container can be sued to perform permission checks, create directories etc.

Eg

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      mountPath: /usr/share/nginx/html
  # These containers are run during pod initialization
  initContainers:
  - name: install
    image: busybox:1.28
    command:
    - wget
    - "-O"
    - "/work-dir/index.html"
    - http://info.cern.ch
    volumeMounts:
    - name: workdir
      mountPath: "/work-dir"
  dnsPolicy: Default
  volumes:
  - name: workdir
    emptyDir: {}
```

# Sidecar Container

A sidecar container is a container that runs alongside the main application container in a pod. It can be used to provide additional functionality, such as logging, monitoring, or proxying.

Eg

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
spec:
  containers:
  - name: main-app
    image: myapp:latest
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/myapp

  - name: log-forwarder
    image: fluentd:latest
    volumeMounts:
    - name: shared-logs
      mountPath: /fluentd/log

  volumes:
  - name: shared-logs
    emptyDir: {}
```

The sidecar container can be used to forward logs from the main application container to a logging service, such as Fluentd or Logstash.


## References

1. https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

2. https://kubernetes.io/docs/tasks/debug/debug-application/debug-init-containers/

