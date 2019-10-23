# Exercise 5: Install the HPE Nimble Volume Driver for Kubernetes FlexVolume Plugin 3.0

>Adapted from Michael Mattsson's post on HPE Storage Tech Insiders: [Released: HPE Volume Driver for Kubernetes FlexVolume Plugin 3.0 and more!](https://community.hpe.com/t5/HPE-Storage-Tech-Insiders/Released-HPE-Volume-Driver-for-Kubernetes-FlexVolume-Plugin-3-0/ba-p/7063875#.Xa87O-hKiUk)

We’ve vastly improved the delivery mechanism of the integrations by enabling Kubernetes administrators to deploy and manage the FlexVolume Driver and Dynamic Provisioner as native Kubernetes workloads. The workloads themselves may be declared directly with kubectl or deployed with Helm — the package manager for Kubernetes applications.

Detailed instructions are available in the [FlexVolume Driver GitHub repository](https://github.com/hpe-storage/flexvolume-driver) on how to deploy the integration either standalone or using Helm.

In this demo, we will go deploy the FlexVolume plugin for Nimble Storage using Helm. Helm is the recommended way to deploy both the FlexVolume and CSI driver for HPE Nimble storage products.

Create a `values.yaml` file:

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
$ helm repo add hpe https://hpe-storage.github.io/co-deployments
"hpe" has been added to your repositories
$ helm install -f values.yaml --name hpe-flexvolume hpe/hpe-flexvolume-driver --namespace kube-system
NAME: hpe-flexvolume
LAST DEPLOYED: Mon Sep 23 11:00:28 2019
NAMESPACE: kube-system
STATUS: DEPLOYED
```

The DaemonSet workload type automatically deploys one replica of the FlexVolume driver on each node of the cluster and ensures all the host packages are installed and necessary services are running prior to serving the cluster. We can see this by querying the DaemonSet.

```
$ kubectl get ds/hpe-flexvolume-driver -n kube-system
NAME                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE  AGE
hpe-flexvolume-driver   3         3         3       3            3          11m
```

Now let's test the deployment by creating a PVC.

Create a `pvc.yaml` file:

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
$ kubectl create -f pvc.yaml
persistentvolumeclaim/my-pvc created
$ kubectl get pvc/my-pvc
NAME   STATUS VOLUME                 CAPACITY ACCESS MODES STORAGECLASS AGE
my-pvc Bound  hpe-standard-9037d5... 32Gi     RWO          hpe-standard 9s
```



**PREVIOUS:** [Exercise 4: Installing Helm](install_helm.md)

**NEXT:** [Exercise 6: Deploying Persistent Apps (Wordpress) with Helm](deploy_app_helm.md)
