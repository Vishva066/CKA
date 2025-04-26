# Persistent Volumes

A PersistentVolume (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.

PVs are volume plugins like Volumes, but have a lifecycle independent of any individual Pod that uses the PV. 

A PersistentVolumeClaim (PVC) is a request for storage by a user. It is similar to a Pod. Pods consume node resources and PVCs consume PV resources. Pods can request specific levels of resources (CPU and Memory). Claims can request specific size and access modes (e.g., they can be mounted ReadWriteOnce, ReadOnlyMany, ReadWriteMany, or ReadWriteOncePod, see AccessModes).

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

