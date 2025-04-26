# ETCD BACKUP AND RESTORE

## ETCD BACKUP AND RESTORE IN THE SAME CLUSTER

### ETCD BACKUP

**Note: Perform these steps from the control pane**


1. Install the etcd client 

```bash

sudo apt install etcd-client

```

2. We need to pass the following three pieces of information to etcdctl to take an etcd snapshot.

- etcd endpoint (--endpoints) (advertise-client-urls)
- ca certificate (--cacert) (cert-file)
- server certificate (--cert) (key-file)
- server key (--key) (trusted-ca-file)

/etc/kubernetes/manifests/etcd.yaml From this location you can get these values.

3. Syntax for snapshot

```bash
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=<ca-file> \
  --cert=<cert-file> \
  --key=<key-file> \
  snapshot save <backup-file-location>
```

4. Command for snapshot

```bash
sudo ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/etcd.db
```

5. Verify the snapshot using this command

```bash
sudo ETCDCTL_API=3 etcdctl --write-out=table snapshot status /opt/etcd.db
```

### ETCD RESTORE

Before rstoring the etcd follow these steps

- Stop all API server instancem

- Restore the etcd instance

- Restart API server instance, kube-scheduler, kube-controller-manager, kubelet

1. Stop the API server instance and other instances

```bash

cd /etc/kubernetes/manifests

sudo mv * /tmp

```

Here we are moving because all these are stored as static pods so it will automatically restart.

2. Restore the etcd instances

```bash
sudo ETCDCTL_API=3 etcdctl --data-dir /opt/etcd snapshot restore /opt/etcd.db

```

3. Now edit the etcd manifest according to this data dir

```bash
volumes:
  - hostPath:
      path: /opt/etcd
      type: DirectoryOrCreate
    name: etcd-data
```

4. Now replace the instances from /tmp so it will automatically restart

```bash
sudo mv /tmp/*.yaml /etc/kubernetes/manifests
```

5. Wait for some time now it will be up and running 