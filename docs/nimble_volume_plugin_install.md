# Exercise 5: Install the HPE Nimble CSI Driver for Kubernetes

HPE Storage has vastly improved the delivery mechanism of the Storage Integrations by enabling Kubernetes administrators to deploy and manage the CSI Driver and CSP as native Kubernetes workloads. The workloads themselves may be declared directly with kubectl via YAML or deployed with Helm — the package manager for Kubernetes applications.

Detailed instructions are available in the [HPE Storage Container Integrations GitHub repository](https://github.com/hpe-storage/co-deployments) on how to deploy the CSI Driver via Helm or using YAML.

In this demo, we will deploy the CSI Driver and CSP for Nimble Storage using Helm. Helm is the recommended way to deploy the CSI drivers for HPE Storage products.

Clone the latest Nimble Helm repo and deploy the HPE Nimble CSI Driver for Kubernetes Helm chart:

```
$ git clone https://github.com/hpe-storage/co-deployments
```

If you see the following error:
```
-bash: git: command not found
```

Run the following to install git:
```
yum install git -y
```

The output is similar to this:
```
$ git clone https://github.com/hpe-storage/co-deployments
$ cd co-deployments/helm/charts/
```

Edit the `values.yaml` file and update the Nimble `backend` array parameter IP. Do not modify anything else.
```
$ vi hpe-csi-driver/values.yaml

***Modify the following section.***

secret:
  # parameters for nimble
  create: true
  backend: <nimble_IP>
  username: admin
  password: admin
  servicePort: "8080"
  serviceName: nimble-csp-svc
  name: nimble-secret
```

Install the Nimble CSI Driver Helm chart.
```
$ helm install nimble-csi hpe-csi-driver --namespace kube-system -f hpe-csi-driver/values.yaml
```

The output is similar to this:
```
$ helm install nimble-csi hpe-csi-driver --namespace kube-system -f hpe-csi-driver/values.yaml
NAME: nimble-csi
LAST DEPLOYED: Mon Mar  2 22:26:34 2020
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

Let's verify everything started up correctly:

```
$ kubectl get pods -n kube-system | grep -e csi -e csp

hpe-csi-controller-667578b549-5k58p        4/4     Running   0          98m
hpe-csi-node-lgrmq                         2/2     Running   0          98m
hpe-csi-node-snmqk                         2/2     Running   0          98m
nimble-csp-b7676477-zhshc                  1/1     Running   0          98m

```

Now let's test the deployment by creating a PVC.

Create a `pvc.yml` file:

```
$ vi pvc.yml
```

Copy and paste the following:
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
```

Create the PVC:

```
$ kubectl create -f pvc.yml
```

The output is similar to this:
```
$ kubectl create -f pvc.yml
persistentvolumeclaim/my-pvc created

# kubectl get pvc
NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mypvc                         Bound    pvc-70d5caf8-7558-40e6-a8b7-77dfcf8ddcd8   16Gi       RWO            hpe-standard   72m
```

We can see the new Persistent Volume created:
```
pvc-70d5caf8-7558-40e6-a8b7-77dfcf8ddcd8
```

We can inspect the PVC further for additional information by using the following commands:

```
$ kubectl describe pvc mypvc
```

The output is similar to this:
```
$ kubectl describe pvc mypvc
Name:          mypvc
Namespace:     default
StorageClass:  hpe-standard
Status:        Bound
Volume:        pvc-70d5caf8-7558-40e6-a8b7-77dfcf8ddcd8
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: csi.hpe.com
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      16Gi
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
$ kubectl describe pvc mypvc
Name:          mypvc
Namespace:     default
StorageClass:  hpe-standard
Status:        Bound
Volume:        pvc-70d5caf8-7558-40e6-a8b7-77dfcf8ddcd8
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: csi.hpe.com
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      16Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Mounted By:    <none>
Events:        <none>
[root@DESKTOP-Q9AGN1K charts]# kubectl describe pv pvc-70d5caf8-7558-40e6-a8b7-77dfcf8ddcd8
Name:            pvc-70d5caf8-7558-40e6-a8b7-77dfcf8ddcd8
Labels:          <none>
Annotations:     pv.kubernetes.io/provisioned-by: csi.hpe.com
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    hpe-standard
Status:          Bound
Claim:           default/mypvc
Reclaim Policy:  Delete
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        16Gi
Node Affinity:   <none>
Message:
Source:
    Type:              CSI (a Container Storage Interface (CSI) volume source)
    Driver:            csi.hpe.com
    VolumeHandle:      0651af878ba1eb8064000000000000000000000001
    ReadOnly:          false
    VolumeAttributes:      accessProtocol=iscsi
                           description=Volume created by the HPE CSI Driver for Kubernetes
                           fsType=xfs
                           storage.kubernetes.io/csiProvisionerIdentity=1583209935353-8081-csi.hpe.com
                           volumeAccessMode=mount
```

With the `describe` command, you can see the volume parameters applied to the volume.

Let's recap what we have learned.

1. We created a default StorageClass for our volumes.
2. We created a PVC that created a volume from the storageClass.
3. We can use **kubectl get** to list the `StorageClass`, `PVC` and `PV`.
4. We can use **kubectl describe** to get details on the `StorageClass`, `PVC` or `PV`

Now we are ready to deploy apps with persistent volumes.


**PREVIOUS:** [Exercise 3: Deploy your first pod](deploy_first_pod.md)

**NEXT:** [Exercise 5: Deploying Persistent Apps (Wordpress) with Helm](deploy_app_helm.md)
