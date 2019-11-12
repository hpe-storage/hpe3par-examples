# Exercise 3: Create a clone of a PVC
The HPE Dynamic Provisioner supports creating clones of an existing `PersistentVolumeClaim` (PVC) in the same namespace.

## Create a PersistentVolumeClaim with an annotation
Creating a clone of a `PVC`. Assume we kept the `PVC` around from the previous exercise.

Create a new `PVC`:
```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc-clone
  annotations:
    hpe.com/cloneOfPVC: mypvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: database
```
Pay attention to the `.metadata.annotations` stanza. It needs to point to the source `PVC` name that already exist in the namespace the new `PVC` is being created into.

### Questions

* After creation, inspect the volume on the underlying array. Is the parent volume what you'd expect?
* If you're using Nimble, if you add `hpe.com/limitIOPS: 1000` to the list of annotations, does it create a throttled clone?

**Important:** Underlying drivers behaves differently. If a `PVC` hasn't been attached to a `Pod`, there might not be a filesystem present on the source volume. This might have odd effects if you're trying to attach the clone to a `Pod` when there's not filesystem. If you intend to attach the clone to a workload in this exercise, make sure the source `PVC` has been successfully attached.

## Alternate methods
It's possible to create a clone using the `cloneOf` and `importVolAsClone` parameter as well. These parameters needs to reference the Docker Volume Name (`cloneOf`) or underlying storage array volume name (`importVolAsClone`). They do provide the benefit that you may operate between namespaces. You may either setup dedicated storage classes where the use case is to clone a production volume into dev/test/staging namespaces or other clusters, or, allow the user to annotate `hpe.com/cloneOf` with the appropriate `allowOverrides: cloneOf`.

## Learn more
Using overrides in the Nimble [docs](https://github.com/hpe-storage/flexvolume-driver/blob/master/USING.md#using-overrides).

* **PREVIOUS**: [Exercise 2: Create a PVC](create_a_pvc.md)
* **NEXT** [Exercise 4: Customized StorageClass and PVC](customize_storageclass.md)
