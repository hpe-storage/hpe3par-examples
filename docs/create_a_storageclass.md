# Exercise 1: Create a StorageClass
Most Kubernetes clusters that have persistent storage capabilities come preconfigured with a default `StorageClass`. In most cases this provide the fundamental ability to satisfy a `PersistentStorageClaim` (PVC) without any particular features or capabilities tailored for a use case or workload that might require a few tweaks.

In this exercise we're going to create a `StorageClass` that has been optmized to run a database workload. Paramters can be tweaked to your liking, the only criteria is that the `StorageClass` needs to be called "database".

### StorageClass parameters
Each platform's parameters differs slightly. 

* [HPE 3PAR/Primera](https://github.com/hpe-storage/python-hpedockerplugin/blob/master/docs/usage.md)
* [HPE Nimble Storage](https://github.com/hpe-storage/flexvolume-driver/tree/master/examples/kubernetes/hpe-nimble-storage)
* [HPE Cloud Volumes](https://github.com/hpe-storage/flexvolume-driver/tree/master/examples/kubernetes/hpe-cloud-volumes)

## StorageClass Overview
A `StorageClass` API object is scoped at the cluster level. Good cluster hygiene is to allow users to read `StorageClass` objects to allow getting an understanding for what storage services are being provided on the cluster. Therefore there shouldn't be any confidential information stored in a `StorageClass`. 

### Nimble Example
The below example creates a `StorageClass` that uses the "SQL Server" Performance Policy on the Nimble array and sets the IOPS limit to 10,000. We'll also allow the user to override the IOPS limitation from `PVC` annotation (explained in a later exercise).

Create the following `StorageClass`:
```yaml
---
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: database
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
provisioner: hpe.com/nimble
parameters:
  allowOverrides: limitIOPS
  limitIOPS: "10000"
  perfPolicy: "SQL Server"
reclaimPolicy: Delete
```
**Gotcha:** A Kubernetes cluster may have multiple storage classes marked "default" but will throw an error if a `PVC` is submitted without an explicit `storageClassName` set.

Storage classes are easiest inspected with:

```
kubectl get sc/database -o yaml
```
Doing a simple `kubectl get sc` will enumerate the storage classes and show which one is the default.

**Note:** "sc" is a valid alias for "storageclass".

## Learn more
Storage Classes on [kubernetes.io](https://kubernetes.io/docs/concepts/storage/storage-classes/)

* **PREVIOUS:** [README.md](README.md)
* **NEXT:** [Exercise 2: Create a PVC](create_a_pvc.md)
