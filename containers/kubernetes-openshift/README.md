## Usage of the HPE 3PAR Volume Plug-in for Docker in Kubernetes/OpenShift<a name="k8_usage"></a>

The following section will cover different operations and commands that can be used to familiarize yourself and verify the installation of the HPE 3PAR Volume Plug-in for Docker by provisioning storage using Kubernetes/OpenShift resources like **PersistentVolume**, **PersistentVolumeClaim**, **StorageClass**, **Pods**, etc.

* [Kubernetes/OpenShift Terms](#terms)
* [StorageClass Example](#sc)
  * [StorageClass options](#sc_parameters)
* [Persistent Volume Claim Example](#pvc)
* [Pod Example](#pod)
* [Restarting the Containerized HPE 3PAR Volume Plug-in](#restart)

To learn more about Persistent Volume Storage and Kubernetes/OpenShift, go to:
https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

### Key Kubernetes/OpenShift Terms:<a name="k8_terms"></a>
* **kubectl** – command line interface for running commands against Kubernetes clusters
* **oc** – command line interface for running commands against OpenShift platform
* **PV** – Persistent Volume is a piece of storage in the cluster that has been provisioned by an administrator.
* **PVC** – Persistent Volume Claim is a request for storage by a user.
* **SC** – Storage Class provides a way for administrators to describe the “classes” of storage they offer.
* **hostPath volume** – mounts a file or directory from the host node’s filesystem into your Pod.

To get started, in an OpenShift environment, we need to relax the security of your cluster so pods are allowed to use the **hostPath** volume plugin without granting everyone access to the privileged **SCC**:

1. Edit the restricted SCC:
```
$ oc edit scc restricted
```

2. Add `allowHostDirVolumePlugin: true`

3. Save the changes

4. Restart node service (master node).
```
$ sudo systemctl restart origin-node.service
```

Below is an example yaml specification to create Persistent Volumes using the **HPE 3PAR FlexVolume driver**. The **HPE 3PAR FlexVolume driver** is a simple daemon that listens for **PVCs** and satisfies those claims based on the defined **StorageClass**.

>Note: If you have OpenShift installed, **kubectl create** and **oc create** commands can be used interchangeably when creating **PVs**, **PVCs**, and **SCs**.

**Dynamic volume provisioning** allows storage volumes to be created on-demand. To enable dynamic provisioning, a cluster administrator needs to pre-create one or more **StorageClass** objects for users. The **StorageClass** object defines the storage provisioner (in our case the **HPE 3PAR Volume Plug-in for Docker**) and parameters to be used when requesting persistent storage within a Kubernetes/Openshift environment. The **StorageClass** acts like a "storage profile" and gives the storage admin control over the types and characteristics of the volumes that can be provisioned within the Kubernetes/OpenShift environment. For example, the storage admin can create multiple **StorageClass** profiles that have size restrictions, if they are full or thin provisioned, compressed, etc.

### StorageClass Example<a name="sc"></a>

The following creates a **StorageClass "sc1"** that provisions a compressed volume with the help of HPE 3PAR Docker Volume Plugin.

```yaml
$ sudo kubectl create -f - << EOF
---
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: sc1
provisioner: hpe.com/hpe
parameters:
  size: "16"
  compression: "true"
EOF
```

#### Supported StorageClass parameters<a name="sc_parameters"></a>

| StorageClass Options | Type    | Parameters                                 | Example                          |
|----------------------|---------|--------------------------------------------|----------------------------------|
| size                 | integer | -                                          | size: "10"                       |
| provisioning         | String  | thin, thick                                | provisioning: thin               |
| flash-cache          | String  | enable, disable                            | flash-cache: enable              |
| compression          | boolean | true, false                                | compression: true                |
| MountConflictDelay   | integer | -                                          | MountConflictDelay: "30"         |
| qos_name             | String  | vvset name                                 | qos_name: "<vvset_name>"         |
| cloneOf              | String  | volume name                                | cloneOf: "<volume_name>"         |
| virtualCopyOf        | String  | volume name                                | virtualCopyOf: "<volume_name>"   |
| expirationHours      | integer | option of virtualCopyOf                    | expirationHours: "10"            |
| retentionHours       | integer | option of virtualCopyOf                    | retentionHours: "10"             |
| accessModes          | String  | ReadWriteOnce                              | accessModes: <br> &nbsp;&nbsp;  - ReadWriteOnce    |


### Persistent Volume Claim Example<a name="pvc"></a>

Now let’s create a claim **PersistentVolumeClaim** (**PVC**). Here we specify the **PVC** name **pvc1** and reference the **StorageClass "sc1"** that we created in the previous step.

```yaml
$ sudo kubectl create -f - << EOF
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: sc1
EOF
```

At this point, after creating the **SC** and **PVC** definitions, the volume hasn’t been created yet. The actual volume gets created on-the-fly during the pod deployment and volume mount phase.

### Pod Example<a name="pod"></a>

So let’s create a **pod "pod1"** using the **nginx** container along with some persistent storage:

```yaml
$ sudo kubectl create -f - << EOF
---
kind: Pod   
apiVersion: v1
metadata:
  name: pod1
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: export
      mountPath: /export
  restartPolicy: Always
  volumes:
  - name: export
    persistentVolumeClaim:
      claimName: pvc1
EOF
```

When the pod gets created and a mount request is made, the volume is now available and can be seen using the following command:

```
$ docker volume ls
DRIVER   VOLUME NAME
hpe      export
```

On the Kubernetes/OpenShift side, it should now look something like this:

```
$ kubectl get pv,pvc,pod -o wide
NAME       CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS    CLAIM            STORAGECLASS   REASON   AGE
pv/pv1     20Gi       RWO           Retain          Bound     default/pvc1                             11m

NAME         STATUS    VOLUME    CAPACITY   ACCESSMODES   STORAGECLASS   AGE
pvc/pvc1     Bound     pv100     20Gi       RWO                          11m

NAME                          READY     STATUS    RESTARTS   AGE       IP             NODE
po/pod1                       1/1       Running   0          11m       10.128.1.53    cld6b16

```

Now the **pod** can be deleted to unmount the Docker volume. Deleting a **Docker volume** does not require manual clean-up because the dynamic provisioner provides automatic clean-up. You can delete the **PersistentVolumeClaim** and see the **PersistentVolume** and **Docker volume** automatically deleted.
