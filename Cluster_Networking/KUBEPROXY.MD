# KUBE PROXY

Kube-proxy is a network component in Kubernetes that manages network rules on nodes. It ensures that network traffic is properly routed to the correct Pods in a cluster. Kube-proxy uses iptables or IPVS to handle traffic routing and load balancing for services.

The Kubernetes network proxy runs on each node.

### Example: Kube-proxy in Action

1. Create a simple deployment:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      replicas: 2
      selector:
         matchLabels:
            app: nginx
      template:
         metadata:
            labels:
              app: nginx
         spec:
            containers:
            - name: nginx
              image: nginx:1.21
              ports:
              - containerPort: 80
    ```

2. Expose the deployment as a service:
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-service
    spec:
      selector:
         app: nginx
      ports:
      - protocol: TCP
         port: 80
         targetPort: 80
      type: ClusterIP
    ```

3. Access the service:
    Use the `nginx-service` ClusterIP to route traffic to the Nginx Pods. Kube-proxy ensures the traffic is load-balanced between the Pods.

This demonstrates how Kube-proxy handles routing and load balancing for services in Kubernetes.

kube-proxy programs the Linux kernel to intercept connections made to the service IP.

Kube-proxy is a pod deployed as a DaemonSet in the cluster and subscribes to changes to Endpoint and Services.

When an Endpoint or Service is created, updated, or deleted, kube-proxy refreshes its internal state for the current node.

kube-proxy primarily sets up iptables rules to route traffic between services and pods on each node to achieve this.

### IP Tables

Iptables is a tool that operates at the network layer. It allows you to configure rules to control incoming and outgoing network traffic.

Iptables is a user-space application that allows you to configure these hooks and create rules to filter and manipulate the traffic.

The most notable example for iptables is firewall rules.

You can define rules to allow or block traffic based on criteria such as source or destination IP address, port, protocol, and more.

A table is a collection of chains that serves a particular function. The 3 main tables in iptables are the Filter, NAT, and Mangle tables.

### IP Tables Chains

| Filter Table       | NAT Table          | Mangle Table       |
|--------------------|--------------------|--------------------|
| INPUT chain        | INPUT chain        | INPUT chain        |
| FORWARD chain      | OUTPUT chain       | FORWARD chain      |
| OUTPUT chain       | PREROUTING chain   | OUTPUT chain       |
| POSTROUTING chain  | POSTROUTING chain  | PREROUTING chain   |
|                    | POSTROUTING chain  |                    |

- The Filter Table is used to control the flow of packets in and out of a system.

- The NAT Table is used to redirect connections to other interfaces on the network.

- The Mangle Table is used to modify packet headers.

### Chains

Chains are a list of rules that are processed in order.

There are 5 main chains in iptables:

- Input: Used to manage incoming packets/connections to service or protocol.

- Output: Outgoing packet after it has been created/processed.

- Forward: Forwards incoming packets from their source to destination (routing).

- Prerouting: After the packet enters the network interface.

- Postrouting: Before the packet leaves the network interface after the routing decision has been made.

### Packet Flow

Incoming packets are analyzed at each chain and are tested against a set of rules. If a rule is matched, the target is set. These are the targets available:

- ACCEPT: Stop processing and lets the packet flow.

- REJECT: Drops the packet with feedback.

- DROP: Stops processing at the current chain and drops the packet.

- LOG: Similar to ACCEPT, however, it is logged to the /var/log/messages.

For example, say you want to block all incoming connections to your web server on port 80. You should add a rule to the incoming chain in the filter table and set the target to REJECT. If the packet reaches the end of the chain without matching to any rules, the default rule is used. If there isn’t a default rule, the packet is accepted.

Example command:

```bash
sudo iptables -t filter -I INPUT -m tcp -p tcp --dport 80 -j REJECT
```

The above command blocks any incoming traffic to the web server/HTTP.

Default policy

```bash
sudo iptables --policy INPUT ACCEPT

sudo iptables --policy FORWARD ACCEPT

sudo iptables --policy OUTPUT ACCEPT
```

You can block all incoming requests from an IP address by adding the following rule:

```bash
sudo iptables -A INPUT -s 192.168.1.1 -j DROP
```

Blocking and allowing connection to ports

```bash
# SSH connection block

sudo iptables -I INPUT -p tcp --dport 22 -j DROP

# Port 80 block

sudo iptables -I INPUT -p tcp --dport 80 -j DROP

# You can go a step further and block a particular IP from connecting to a particular service. For example, you can block an IP from accessing our web server by adding the following rule:

sudo iptables -I INPUT -p tcp --dport 80 -s <IP> -j DROP
```

### References

1. https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/?ref=devopscube.com

2. https://www.linode.com/docs/guides/what-is-iptables/
