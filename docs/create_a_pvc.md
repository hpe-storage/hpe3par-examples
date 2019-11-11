# Exercise 2: Create a PVC
In this exercise we'll create a `PersistentVolumeClaim` (PVC) using the `StorageClass` we created in the previous exercise. We'll also inspect some of the properties in this objects created.

## PersistenVolumeClaim
A PVC is submitted to the cluster in an attempt to request a `PersistentVolume` matching the criteria requested. In most basic example, only capacity is requested. Specifying capacity is also mandatory.

Create the following `PVC`:
```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 16Gi
  storageClassName: database
```

Looking at the `storageClassName` parameter, we're assuming that we have a `StorageClass` named "database" that is capable of dynamic provisioning.

## Inspecting resources
Immediately after a `PVC` creation, the `PVC` will be set to "Pending", usually it just takes a few seconds before it is "Bound". It will be stuck "Pending" if the provisioner is unable to create the `PersistentVolume` and bind it to the `PVC`.

Inspect the `PV` and `PVC`:
```
kubectl get pvc/mypvc -o yaml
kubectl get pv/database-UUID -o yaml
```

Inspect the `flexVolume` stanza. Is this what you'd expect comparing the `StorageClass`?

**Good to know:** Lookup what `.metadata.finalizers` do. Both `kubernetes.io/pv-protection` and `kubernetes.io/pvc-protection`. 

## Learn more
Dynamic Volume Provisioning on [kubernetes.io](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/).

**PREVIOUS:** [Exercise 1: Create a StorageClass](create_a_storageclass.md)
**NEXT:** [Exercise 3: Create a Clone of PVC](create_a_cloneofpvc.md)
