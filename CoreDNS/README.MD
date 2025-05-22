# COREDNS

CoreDNS is a flexible, extensible DNS server that can serve as the Kubernetes cluster DNS

CoreDNS server can be configured by maintaining a Corefile, which is the CoreDNS configuration file. 

As a cluster administrator, you can modify the ConfigMap for the CoreDNS Corefile to change how DNS service discovery behaves for that cluster.

Whenever a new pod is getting created a dns resolve file will be created (/etc/resolv.conf) in that pod there the **nameserver IP will point to the service IP of the coredns**

Core DNS works only when Network Operator is installed (Calico)

Eg of coredns configmap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health {
            lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
            pods insecure
            fallthrough in-addr.arpa ip6.arpa
            ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }    
```

### Explaination of the Corefile(ConfigMap)

1. Zone Definition (.:53)
.: Represents the root zone, meaning CoreDNS will handle all DNS queries.
:53: Specifies that CoreDNS listens on port 53, the default DNS port.

2. errors
Logs errors to the standard output, useful for debugging DNS issues.

3. health
Provides a health check endpoint for CoreDNS.
lameduck 5s: Waits 5 seconds before shutting down when CoreDNS is terminated, allowing ongoing requests to complete.

4. ready
Exposes a readiness endpoint to indicate when CoreDNS is ready to serve DNS queries

5. kubernetes
Configures CoreDNS to handle DNS queries for Kubernetes services and pods.
- cluster.local: The default DNS domain for the Kubernetes cluster.

- in-addr.arpa and ip6.arpa: Handle reverse DNS lookups for IPv4 and IPv6 addresses.

- pods insecure: Allows DNS queries for pod IPs without authentication.

- fallthrough in-addr.arpa ip6.arpa: Passes unresolved reverse DNS queries to the next plugin.

- ttl 30: Sets the time-to-live (TTL) for DNS responses to 30 seconds.

6. prometheus :9153
Exposes CoreDNS metrics on port 9153 for monitoring via Prometheus.

7. forward . /etc/resolv.conf
Forwards unresolved DNS queries to the upstream DNS servers specified in resolv.conf.

8. cache 30
Caches DNS responses for 30 seconds to improve performance and reduce upstream queries.

9. loop
Detects and prevents infinite DNS query loops.

10. reload
Automatically reloads the Corefile when changes are detected.

11. loadbalance
Balances DNS responses across multiple upstream servers to distribute traffic evenly.

**Summary:**
This Corefile configures CoreDNS to:

- Serve as the DNS server for the Kubernetes cluster (kubernetes plugin).

- Forward unresolved queries to upstream DNS servers (forward plugin).

- Provide health and readiness endpoints.

- Cache responses for performance.

- Expose metrics for monitoring.

- Handle reverse DNS lookups and balance traffic efficiently.

## Example Questions

**Q.** Configuren CoreDns such that any dns query for 'myapp.internal' returns the following IP 10.10.10.10

After configuration pods should be able to resolve myapp.internal to 10.10.10.10.

**Ans:**

Add the following in core file (config map)

hosts{
  myapp.internal 10.10.10.10
  fallthrough # If it fails pass the to next one
}

### References

1. https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/
