# Storage Classes

A StorageClass provides a way for administrators to describe the classes of storage they offer.

Each StorageClass contains the fields provisioner, parameters, and reclaimPolicy, which are used when a PersistentVolume belonging to the class needs to be dynamically provisioned to satisfy a PersistentVolumeClaim (PVC).


In Kubernetes, when you create a PersistentVolumeClaim (PVC), it usually needs to say what type of storage it wants — like SSD, standard disk, etc.

But sometimes, developers might not mention which storage class they want.

So as an admin, you can set a default StorageClass — like a default storage plan. That way, if someone forgets to specify one, Kubernetes still knows what type of storage to use.

Eg of Storage Class

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: low-latency
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
provisioner: csi-driver.example-vendor.example
reclaimPolicy: Retain # default value is Delete
allowVolumeExpansion: true
mountOptions:
  - discard # this might enable UNMAP / TRIM at the block storage layer
volumeBindingMode: WaitForFirstConsumer
parameters:
  guaranteedReadWriteLatency: "true" # provider-specific
```

When a PVC does not specify a storageClassName, the default StorageClass is used.

## Reclaim Policy

PersistentVolumes that are dynamically created by a StorageClass will have the reclaim policy specified in the reclaimPolicy field of the class, which can be either Delete or Retain. If no reclaimPolicy is specified when a StorageClass object is created, it will default to Delete.


