# Exercise 5: Install the HPE 3PAR/Primera CSI Driver

## This page will be updated upon the release of the HPE 3PAR/Primera CSI Driver. Stay Tuned!

For information on how to configure the HPE 3PAR/Primera FlexVolume Driver for Kubernetes. [Install the HPE 3PAR FlexVolume Driver for Kubernetes](3par_volume_plugin_install_flexvolume.md)

```

Once complete you will be ready to start using the HPE 3PAR Volume Plug-in for Docker.

### Now let's create a StorageClass

Creating a default StorageClass

```yaml
kubectl create -f - << EOF
---
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: sc-basic
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: hpe.com/hpe
parameters:
  provisioning: 'full'
EOF
```

Available Storage Class options:
[StorageClass Options](https://community.hpe.com/t5/HPE-Storage-Tech-Insiders/How-to-Persistent-Volumes-with-the-HPE-3PAR-Volume-Plug-in-for/ba-p/7060077#.XcNQd-hKguU)

A `PersistentVolumeClaim` (PVC) is a request for storage by a user. It is based off of a StorageClass.

For example, we will be requesting a 250 GB volume based on the StorageClass `sc-basic`. Also because we are requesting block storage, we will need to need to specify ReadWriteOnce for the AccessModes.

```yaml
kubectl create -f - << EOF
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-basic
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 32Gi
  storageClassName: sc-basic
EOF
```

Use the **kubectl get pvc** command to view the PVC

The output is similar to this:
```
$ kubectl get pvc
NAME      STATUS  VOLUME                                         CAPACITY  ACCESS MODES  STORAGECLASS   AGE
pvc-basic Bound   sc-basic-b9e47260-045f-11ea-aaa4-0050569bb07c0  32Gi     RWO           sc-basic       6s
```

We can see the new Persistent Volume created:
```
sc-basic-b9e47260-045f-11ea-aaa4-0050569bb07c
```

We can inspect the PVC further for additional information by using the following commands:

```
$ kubectl describe pvc pvc-basic
```
The output is similar to this:
```
$ kubectl describe pvc pvc-basic
Name:          pvc-basic
Namespace:     default
StorageClass:  sc-basic
Status:        Bound
Volume:        sc-basic-b9e47260-045f-11ea-aaa4-0050569bb07c
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed=yes
               pv.kubernetes.io/bound-by-controller=yes
               volume.beta.kubernetes.io/storage-provisioner=hpe.com/hpe
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      32Gi
Access Modes:  RWO
Events:
  Type    Reason                Age                From                         Message
  ----    ------                ----               ----                         -------
  Normal  ExternalProvisioning  32s (x2 over 32s)  persistentvolume-controller  waiting for a volume to be created, either by external provisioner "hpe.com/hpe" or manually created by system administrator
```

We can also inspect the volume in a similar manner:
```
$ kubectl describe pv sc-basic-66d57a3e-6345-4c7f-a875-f3620b1274a0
```

The output is similar to this:
```
# oc describe pv sc-basic-b9e47260-045f-11ea-aaa4-0050569bb07c
Name:            sc-basic-b9e47260-045f-11ea-aaa4-0050569bb07c
Labels:          <none>
Annotations:     hpe.com/docker-volume-name=sc-basic-b9e47260-045f-11ea-aaa4-0050569bb07c
                 pv.kubernetes.io/provisioned-by=hpe.com/hpe
                 volume.beta.kubernetes.io/storage-class=sc-basic
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    sc-basic
Status:          Bound
Claim:           default/pvc-basic
Reclaim Policy:  Delete
Access Modes:    RWO
Capacity:        32Gi
Node Affinity:   <none>
Message:
Source:
    Type:       FlexVolume (a generic volume resource that is provisioned/attached using an exec based plugin)
    Driver:     hpe.com/hpe
    FSType:
    SecretRef:  <nil>
    ReadOnly:   false
    Options:    map[name:sc-basic-b9e47260-045f-11ea-aaa4-0050569bb07c provisioning:full]
Events:         <none>
```

With the describe command you can see the volume parameters applied to the volume.

Let's recap what we have learned.

1. We created a default StorageClass for our volumes.
2. We created a PVC that created a volume from the storageClass.
3. We can use **kubectl get** to list the `StorageClass`, `PVC` and `PV`.
4. We can use **kubectl describe** to get details on the `StorageClass`, `PVC` or `PV`

Now we are ready to deploy apps with persistent volumes.


**PREVIOUS:** [Exercise 4: Installing Helm](install_helm.md)

**NEXT:** [Exercise 6: Deploying Persistent Apps (Wordpress) with Helm](deploy_app_helm.md)
