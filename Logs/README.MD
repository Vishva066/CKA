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

To filter logs or output for a particular section using `grep`, you can use:

```bash
grep "desired_word" logs.txt
```

To view five lines after the matched word, use the `-A` option:

```bash
grep -A 5 "desired_word" logs.txt
```

This will display the matched line and the next five lines following it.