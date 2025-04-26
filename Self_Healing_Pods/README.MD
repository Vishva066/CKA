## POD LIFECYCLE

THere are different pod phases they are 

- Pending -> The Pod has been accepted by the Kubernetes cluster, but one or more of the containers has not been set up and made ready to run. This includes time a Pod spends waiting to be scheduled as well as the time spent downloading container images over the network.

- Running -> The Pod has been bound to a node, and all of the containers have been created. At least one container is still running, or is in the process of starting or restarting.

- Succeeded -> All containers in the Pod have terminated in success, and will not be restarted.

- Failed -> All containers in the Pod have terminated, and at least one container has terminated in failure. That is, the container either exited with non-zero status or was terminated by the system, and is not set for automatic restarting.

- Unknown -> For some reason the state of the Pod could not be obtained. This phase typically occurs due to an error in communicating with the node where the Pod should be running.

---

**How Pod handles problems ?**

- Initial Crash ->  Kubernetes attempts an immediate restart based on the Pod restartPolicy.

- Repeated crashes -> After the initial crash Kubernetes applies an exponential backoff delay for subsequent restarts, described in restartPolicy. This prevents rapid, repeated restart attempts from overloading the system.

- CrashLoopBackOff -> A container in the Pod is repeatedly crashing. Kubernetes will try to restart it, but if it keeps failing, it will back off and wait longer between attempts.

- Backoff reset -> If a container runs successfully for a certain duration (e.g., 10 minutes), Kubernetes resets the backoff delay, treating any new crash as the first one.

## Pod Probes

- live-ness probe -> Kubernetes uses this to check if the application inside the container is still running. If the probe fails, Kubernetes kills the container and restarts it.

- readiness probe -> Kubernetes uses this to check if the application inside the container is ready to accept traffic. If the probe fails, Kubernetes marks the Pod as not ready, and it will not receive any traffic until it becomes ready again.

- startup probe -> This is used to check if the application inside the container has started. If the probe fails, Kubernetes kills the container and restarts it. This is useful for applications that take a long time to start.


## live-ness probe

When to use liveness probe

- Your app might hang or freeze, but the container is still technically running.

- You want Kubernetes to self-heal your pod by restarting it if it’s no longer healthy.

Eg

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-http-example
spec:
  containers:
  - name: myapp
    image: myapp:latest
    ports:
    - containerPort: 8080
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
```

- httpGet: Kubernetes will send an HTTP GET request to /healthz on port 8080.

- initialDelaySeconds: 5: Wait 5 seconds after the container starts before checking.

- periodSeconds: 10: Check every 10 seconds.

If the /healthz endpoint doesn't return a 200-299 status code, it's considered failed, and Kubernetes will restart the container.

Besides httpGet, you can also use other types of probes:

Type | What it does
exec | Runs a command inside the container (e.g. cat /tmp/healthy)
tcpSocket | Checks if a TCP port is open and responding

exec eg 

```yaml
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 10
```

Kubernetes will check if the file /tmp/healthy exists — if not, it restarts the container.



## readiness probe

When to use readiness probe

- Your app is not ready to accept traffic, but the container is running.
- You want Kubernetes to manage the traffic to your pod based on its readiness.
- You want to avoid sending traffic to a pod that is still starting up or initializing.



Eg

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-http-example
spec:
  containers:
  - name: myapp
    image: myapp:latest
    ports:
    - containerPort: 8080
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10

```

Kubernetes sends an HTTP request to /ready on port 8080.

Starts checking after 5 seconds.

Checks every 10 seconds.

If it doesn’t get a 200–299 HTTP response, the pod is marked as Not Ready, and the Service will stop sending traffic to it.

## startup probe

When to use startup probe

- Your app takes a long time to start up, and you want to avoid the liveness probe killing it during that time.
- You want to separate the startup check from the liveness check, allowing for different configurations.
- You want to ensure that your app is fully initialized before Kubernetes starts checking its health.

Eg

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startup-probe-example
spec:
  containers:
  - name: myapp
    image: myapp:latest
    ports:
    - containerPort: 8080
    startupProbe:
      httpGet:
        path: /healthz
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      periodSeconds: 5
```

Startup Probe:

It checks /healthz every 10 seconds.

Will try up to 30 times (i.e., up to 5 minutes).

If it fails for all 30 tries, the container is restarted.

If it succeeds, liveness and readiness probes begin checking.

### References

https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/

