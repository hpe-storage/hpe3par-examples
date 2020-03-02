# Exercise 5: Install the HPE Nimble Volume Driver for Kubernetes FlexVolume Plugin 3.0

>Adapted from Michael Mattsson's post on HPE Storage Tech Insiders: [Released: HPE Volume Driver for Kubernetes FlexVolume Plugin 3.0 and more!](https://community.hpe.com/t5/HPE-Storage-Tech-Insiders/Released-HPE-Volume-Driver-for-Kubernetes-FlexVolume-Plugin-3-0/ba-p/7063875#.Xa87O-hKiUk)


We’ve vastly improved the delivery mechanism of the integrations by enabling Kubernetes administrators to deploy and manage the FlexVolume Driver and Dynamic Provisioner as native Kubernetes workloads. The workloads themselves may be declared directly with kubectl or deployed with Helm — the package manager for Kubernetes applications.

Detailed instructions are available in the [FlexVolume Driver GitHub repository](https://github.com/hpe-storage/flexvolume-driver) on how to deploy the integration either standalone or using Helm.

In this demo, we will go deploy the FlexVolume plugin for Nimble Storage using Helm. Helm is the recommended way to deploy both the FlexVolume and CSI driver for HPE Nimble storage products.

Create a `values.yml` file:

```
nano values.yml
```

Copy and paste the following:
```yaml
---
backend: "<nimble_IP>"
username: "admin"
password: "admin"
pluginType: "nimble"
protocol: "iscsi"
fsType: "xfs"
storageClass:
  create: true
  defaultClass: true
```

Add Helm repo and deploy the HPE Nimble Volume Driver for Kubernetes FlexVolume Plugin:

```
helm repo add hpe https://hpe-storage.github.io/co-deployments
```

The output is similar to this:
```
$ helm repo add hpe https://hpe-storage.github.io/co-deployments
"hpe" has been added to your repositories
$ helm install -f values.yml --name hpe-flexvolume hpe/hpe-flexvolume-driver --version 3.0.0 --namespace kube-system
NAME: hpe-flexvolume
LAST DEPLOYED: Mon Sep 23 11:00:28 2019
NAMESPACE: kube-system
STATUS: DEPLOYED
```

The `DaemonSet` workload type automatically deploys one replica of the FlexVolume driver on each node of the cluster and ensures all the host packages are installed and necessary services are running prior to serving the cluster. We can see this by querying the `DaemonSet`.

```
kubectl get ds/hpe-flexvolume-driver -n kube-system
```

The output is similar to this:
```
$ kubectl get ds/hpe-flexvolume-driver -n kube-system
NAME                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE  AGE
hpe-flexvolume-driver   3         3         3       3            3          11m
```

Now let's test the deployment by creating a PVC.

Create a `pvc.yml` file:

```
nano pvc.yml
```

Copy and paste the following:
```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 32Gi
```

Create the PVC:

```
kubectl create -f pvc.yml
```

The output is similar to this:
```
$ kubectl create -f pvc.yml
persistentvolumeclaim/my-pvc created
$ kubectl get pvc/my-pvc
NAME   STATUS VOLUME                 CAPACITY ACCESS MODES STORAGECLASS AGE
my-pvc Bound  hpe-standard-9a87fb... 32Gi     RWO          hpe-standard 9s
```

We can see the new Persistent Volume created:
```
hpe-standard-9a87fb4a-4b3b-441c-be46-7fc1b7cb5c4b
```

We can inspect the PVC further for additional information by using the following commands:

```
$ kubectl describe pvc my-pvc
```

The output is similar to this:
```
kubectl describe pvc my-pvc
Name:          my-pvc
Namespace:     default
StorageClass:  hpe-standard
Status:        Bound
Volume:        hpe-standard-9a87fb4a-4b3b-441c-be46-7fc1b7cb5c4b
Labels:        <none>
Annotations:   kubectl.kubernetes.io/last-applied-configuration:
                 {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"my-pvc","namespace":"default"},"spec":{"accessModes...
               pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: hpe.com/nimble
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      32Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Mounted By:    <none>
Events:        <none>
```

We can also inspect the volume in a similar manner:
```
$ kubectl describe pv hpe-standard-9a87fb4a-4b3b-441c-be46-7fc1b7cb5c4
```

The output is similar to this:
```
kubectl describe pv hpe-standard-9a87fb4a-4b3b-441c-be46-7fc1b7cb5c4b
Name:            hpe-standard-9a87fb4a-4b3b-441c-be46-7fc1b7cb5c4b
Labels:          <none>
Annotations:     hpe.com/docker-volume-name: hpe-standard-9a87fb4a-4b3b-441c-be46-7fc1b7cb5c4b
                 pv.kubernetes.io/provisioned-by: hpe.com/nimble
                 volume.beta.kubernetes.io/storage-class: hpe-standard
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    hpe-standard
Status:          Bound
Claim:           default/my-pvc
Reclaim Policy:  Delete
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        32Gi
Node Affinity:   <none>
Message:
Source:
    Type:       FlexVolume (a generic volume resource that is provisioned/attached using an exec based plugin)
    Driver:     hpe.com/nimble
    FSType:
    SecretRef:  nil
    ReadOnly:   false
    Options:    map[description:Volume created by HPE Volume Driver for Kubernetes FlexVolume Plugin name:hpe-standard-9a87fb4a-4b3b-441c-be46-7fc1b7cb5c4b]
Events:         <none>
```

With the describe command  you can see the volume parameters applied to the volume.

Let's recap what we have learned.

1. We created a default StorageClass for our volumes.
2. We created a PVC that created a volume from the storageClass.
3. We can use **kubectl get** to list the `StorageClass`, `PVC` and `PV`.
4. We can use **kubectl describe** to get details on the `StorageClass`, `PVC` or `PV`

Now we are ready to deploy apps with persistent volumes.


**PREVIOUS:** [Exercise 3: Deploy your first pod](deploy_first_pod.md)

**NEXT:** [Exercise 5: Deploying Persistent Apps (Wordpress) with Helm](deploy_app_helm.md)
