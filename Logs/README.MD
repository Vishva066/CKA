If you want to redirect the logs to a particular file use this command

```bash
kubectl logs <pod-name> > logs.txt
```

This command > only captures the stdout

To capture both stdout and stderr use this command 2>&1

```bash
ETCDCTL_API=3 etcdctl --data-dir /root/default.etcd  snapshot restore /opt/cluster_backup.db > restore.txt 2>&1
```

To get the value using json path

```bash
kubectl get svc redis-server --jsonpath='{.spec.ports[0].targetport}'
```