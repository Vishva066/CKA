# System Capabilites

In Kubernetes, system capabilities (or Linux capabilities) refer to a subset of the traditional root privileges that can be individually assigned or removed from processes in containers.

## Capabilities

Capability | Description
NET_ADMIN | Allows changing network interfaces and IP routing
SYS_TIME | Allows changing system clock
SYS_ADMIN | Broad, very powerful — allows mounting, setting hostname, etc. (almost root)
CHOWN | Allows changing ownership of files
MKNOD | Allows creating special files
AUDIT_WRITE | Allows writing to audit logs


Eg

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      capabilities:
        add: ["NET_ADMIN"]
        drop: ["ALL"]
```