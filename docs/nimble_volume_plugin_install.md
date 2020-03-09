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
```yaml
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

Wait while the plugin deploys, this can take a few minutes. We can verify everything started up correctly:

```
$ kubectl get pods -n kube-system | grep -e csi -e csp

hpe-csi-controller-667578b549-5k58p        4/4     Running   0          98m
hpe-csi-node-lgrmq                         2/2     Running   0          98m
hpe-csi-node-snmqk                         2/2     Running   0          98m
nimble-csp-b7676477-zhshc                  1/1     Running   0          98m

```

With the plugin deployed, we can look at the StorageClass that was created as part of the deployment.

```
$ kubectl get sc
NAME           PROVISIONER   AGE
hpe-standard   csi.hpe.com   5m

```

Now we will create our own StorageClass that we will use in the following exercises. Remember a `StorageClass` is similar to a Storage Profile and specifies the characteristics (such as Protection Templates, Performance Policies, etc) of the volume that we want to create.

Create a `sc-gold.yml` file

```
$ vi sc-gold.yml
```

Copy and paste the following:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: sc-gold
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
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
  performancePolicy: "SQL Server"
  description: "Volume from HPE CSI Driver"
  accessProtocol: "iscsi"
  limitIops: "76800"
  allowOverrides: description,limitIops,performancePolicy
```  

Save and exit.

Create the StorageClass:
```
$ kubectl create sc-gold.yml
```

Now let's look at the available StorageClasses.

```
$ kubectl get sc
NAME                PROVISIONER   AGE
hpe-standard        csi.hpe.com   23h
sc-gold (default)   csi.hpe.com   23s
```

>**Note:** We set `sc-gold` StorageClass as default using `storageclass.kubernetes.io/is-default-class: "true"`
>To learn more about configuring a default StorageClass.
>[Default StorageClass](default_storageclass.md)

We are now ready to creating a PVC.

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
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 16Gi
#  storageClassName: hpe-standard
```
>**Note:** We can use `storageClassName` to override the default `StorageClass` with another available `StorageClass`.

Create the PVC:

```
$ kubectl create -f pvc.yml
```

The output is similar to this:
```
$ kubectl create -f pvc.yml
persistentvolumeclaim/my-pvc created

$ kubectl get pvc
NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
my-pvc                         Bound    pvc-70d5caf8-7558-40e6-a8b7-77dfcf8ddcd8   16Gi       RWO            hpe-standard   72m
```

>If you see the PVC status stuck in **Pending**, use `kubectl describe pvc my-pvc` to find the error. Verify that the StorageClass is correctly defined.

We can see the new Persistent Volume created:
```
pvc-70d5caf8-7558-40e6-a8b7-77dfcf8ddcd8
```

We can inspect the PVC further for additional information by using the following commands:

```
$ kubectl describe pvc my-pvc
```

The output is similar to this:
```yaml
$ kubectl describe pvc my-pvc
Name:          my-pvc
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

We can also inspect the volume in a similar manner:
```
$ kubectl describe pv pvc-70d5caf8-7558-40e6-a8b7-77dfcf8ddcd8
```

The output is similar to this:
```yaml
$ kubectl describe pv pvc-659a82a6-98bd-49c1-bf2e-6bcea66ae03a
Name:            pvc-659a82a6-98bd-49c1-bf2e-6bcea66ae03a
Labels:          <none>
Annotations:     pv.kubernetes.io/provisioned-by: csi.hpe.com
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    sc-gold
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
    VolumeHandle:      063aba3d50ec99d866000000000000000000000001
    ReadOnly:          false
    VolumeAttributes:      accessProtocol=iscsi
                           allowOverrides=description,limitIops,performancePolicy
                           description=Volume from HPE CSI Driver
                           fsType=xfs
                           limitIops=76800
                           performancePolicy=SQL Server
                           storage.kubernetes.io/csiProvisionerIdentity=1583271972595-8081-csi.hpe.com
                           volumeAccessMode=mount
Events:                <none>
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
