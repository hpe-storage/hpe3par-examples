# Exercise 4: Customized StorageClass and PVC
Any HPE storage platform allows for endless customization. It's considered best practice to have a default "catch all" `StorageClass` that fulfill the lowest common denominator for the SLO/SLA that the cluster is supposed to provide. RPO/RTO should also be considered. 

In this exercise we'll walk through an example `StorageClass` that would make best use of the Nimble parameters. For 3PAR/Primera users, change parameters according to the [3PAR/Primera documentation](https://github.com/hpe-storage/python-hpedockerplugin/blob/master/docs/usage.md).

## Use case definition
Good practice is to try define in plain English what the `StorageClass` objectives are in terms of "user stories".

* As Kube Admin I need to provide basic service for stateful applications while meeting SLA/SLO and RPO/RTO without obstructing users from supporting themselves.
* As Kube User I don't want to care about persistent storage unless it's a key workload that needs certain tweaks to perform well and be available off-site.

From those two statements it seems we can derive a plan. Let's assume this `StorageClass`:

```yaml
---
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: custom
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
provisioner: csi.hpe.com
parameters:
  csi.storage.k8s.io/fstype: xfs
  csi.storage.k8s.io/provisioner-secret-name: nimble-secret
  csi.storage.k8s.io/provisioner-secret-namespace: kube-system
  csi.storage.k8s.io/controller-publish-secret-name: nimble-secret
  csi.storage.k8s.io/controller-publish-secret-namespace: kube-system
  csi.storage.k8s.io/node-stage-secret-name: nimble-secret
  csi.storage.k8s.io/node-stage-secret-namespace: kube-system
  csi.storage.k8s.io/node-publish-secret-name: nimble-secret
  csi.storage.k8s.io/node-publish-secret-namespace: kube-system
  csi.storage.k8s.io/controller-expand-secret-name: nimble-secret
  csi.storage.k8s.io/controller-expand-secret-namespace: kube-system
  accessProtocol: "iscsi"
  allowOverrides: "limitIOPS,perfPolicy,protectionTemplate,destroyOnRm,thick"
  limitIOPS: "5000"
  perfPolicy: "Retain-90Daily"
```

This storage class meets the following objectives:

* Allow users to control performance through IOPS and Performance Policy.
* Ensures there's at least snapshots on the volumes and that volumes are not deleted when fat fingered.

Shifting to the user role, I can now annotate my way throgh a series of use cases.

### High performance mission-critical database

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: hpdb
  annotations:
    csi.hpe.com/description: "This is my custom db description"
    csi.hpe.com/limitIops: "8000"
    csi.hpe.com/performancePolicy: "Oracle OLTP"
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: custom
```

### General purpose unstructured storage

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: gpf
  annotations:
    csi.hpe.com/limitIOPS: "5000"
    csi.hpe.com/perfPolicy: "Windows File Server"
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
  storageClassName: custom
```
### Disposable clones

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: clone
  annotations:
    csi.hpe.com/limitIOPS: "1000"
    csi.hpe.com/destroyOnRm: "true"
    csi.hpe.com/cloneOfPVC: "hpdb"
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: custom
```
**Note:** For the clone to work, the parent `PVC` needs to have been mounte prior to creation. See "Appendix" below.

In these generic use cases, we can use a single `StorageClass` to cater to many different workloads.

### Questions

* When you delete a custom volume vs delete the clone. What can you observe on the array?
* What other use cases can you think of by looking at the platform capabilities?

## Learn more

* [HPE 3PAR/Primera](https://github.com/hpe-storage/python-hpedockerplugin/blob/master/docs/usage.md)
* [HPE Nimble Storage and HPE Cloud Volumes](https://github.com/hpe-storage/flexvolume-driver/blob/master/USING.md)

* **PREVIOUS:** [Exercise 3: Create a Clone of PVC](create_a_cloneofpvc.md)
* **NEXT:** [Exercise 5: Create a StatefulSet](create_a_statefulset.md)

# Appendix

A simple `Pod` that attaches the "hpdb" `PVC`:
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
    volumeMounts:
      - mountPath: /data
        name: data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: hpdb
```
