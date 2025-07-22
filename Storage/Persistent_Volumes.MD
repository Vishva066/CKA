# Persistent Volumes

A PersistentVolume (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.

PVs are volume plugins like Volumes, but have a lifecycle independent of any individual Pod that uses the PV. 

A PersistentVolumeClaim (PVC) is a request for storage by a user. It is similar to a Pod. Pods consume node resources and PVCs consume PV resources. Pods can request specific levels of resources (CPU and Memory). Claims can request specific size and access modes (e.g., they can be mounted ReadWriteOnce, ReadOnlyMany, ReadWriteMany, or ReadWriteOncePod, see AccessModes).

A PersistentVolume (PV) is not namespaced but a PersistentVolumeClaim (PVC) is namespaced.

- If a user tries to delete a PersistentVolumeClaim (PVC) that is still being used by a Pod, it won't be deleted right away. The system waits until the Pod stops using it.

- Similarly, if an admin tries to delete a PersistentVolume (PV) that is linked to a PVC, it won't be deleted immediately. The system waits until the PV is no longer linked to any PVC.

In short, the system ensures that storage resources (PVCs and PVs) are not removed while they are still in use, to avoid breaking anything.



## Lifecycle of a PV

- Provising 

    - Static: Manually created by administrators.
    
    - Dynamic: Automatically created by Kubernetes when a PVC requests storage and no suitable PV exists. This requires a StorageClass to be defined.

- Binding: Kubernetes matches a PVC to an appropriate PV based on size, access modes, and other criteria. Once bound, the PV is exclusively used by the PVC.

- Using: Pods use PVCs as volumes. The cluster inspects the PVC to find the bound PV and mounts that volume for the Pod.

- Reclaiming: Determines what happens to a PV after its PVC is deleted. Policies include:

    - Retain: PV remains for manual reclamation.

    - Delete: PV and associated storage are deleted.

    - Recycle: (Deprecated) Performs a basic scrub on the volume.

## Retain

The Retain reclaim policy allows for manual reclamation of the resource. When the PersistentVolumeClaim is deleted, the PersistentVolume still exists and the volume is considered "released". 

But it is not yet available for another claim because the previous claimant's data remains on the volume. An administrator can manually reclaim the volume with the following steps.

Delete the PersistentVolume. The associated storage asset in external infrastructure still exists after the PV is deleted.

Manually clean up the data on the associated storage asset accordingly.
Manually delete the associated storage asset.

If you want to reuse the same storage asset, create a new PersistentVolume with the same storage asset definition.

## Delete

For volume plugins that support the Delete reclaim policy, deletion removes both the PersistentVolume object from Kubernetes, as well as the associated storage asset in the external infrastructure. 

Volumes that were dynamically provisioned inherit the reclaim policy of their StorageClass, which defaults to Delete.



## Accesss Modes

- ReadWriteOnce (RWO): Mounted as read-write by a single node.

- ReadOnlyMany (ROX): Mounted as read-only by many nodes.

- ReadWriteMany (RWX): Mounted as read-write by many nodes.

- ReadWriteOncePod (RWOP): Mounted as read-write by a single Pod

## Volume Modes:

- Filesystem: Default mode, mounts the volume as a filesystem.

- Block: Exposes the volume as a raw block device.​

# Persistent Volume Claim

A PVC is a request for storage by a user. It specifies size, access modes, and optionally, a StorageClass.

Kubernetes binds the PVC to a suitable PV. If none is available, and dynamic provisioning is enabled, a new PV is created.​

Volume Binding: PVCs can specify a particular PV by name using the volumeName field.​
Kubernetes

Selectors: PVCs can use label selectors to match specific PVs.​


## PV Example

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: foo-pv
spec:
  storageClassName: ""
  claimRef:
    name: foo-pvc
    namespace: foo
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2

```
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: demo-pv
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  storageClassName: standard
  hostPath:
    path: /tmp/demo-pv
```

## PVC Example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: foo-pvc
  namespace: foo
spec:
  storageClassName: "" # Empty string must be explicitly set otherwise default StorageClass will be set
  volumeName: foo-pv
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard
  volumeName: demo-pv
```


## Expanding PVC

CSI based volume can be expanded 

A PVC can be expanded only when its storage class's allowVolumeExpansion field is set to true.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: example-vol-default
provisioner: vendor-name.example/magicstorage
parameters:
  resturl: "http://192.168.10.100:8080"
  restuser: ""
  secretNamespace: ""
  secretName: ""
allowVolumeExpansion: true
```

## Volume Claim

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
    - name: pvc-pod-container
      image: nginx:latest
      volumeMounts:
        - mountPath: /data
          name: data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: pvc-dynamic
```