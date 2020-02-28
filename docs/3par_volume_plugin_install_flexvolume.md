# Exercise 5: Install the HPE 3PAR Volume Driver for Docker version 3.3

Adapted from the official HPE 3PAR Volume Driver documentation found on their Github page: [HPE 3PAR and Primera Volume Plugin for Docker](https://github.com/hpe-storage/python-hpedockerplugin/)

## Automated Installer for 3PAR Docker Volume plugin (Ansible)
These are Ansible playbooks to automate the install of the HPE 3PAR Volume Plug-in for Docker for use within Kubernetes/OpenShift environments.

### Getting Started
These playbooks perform the following tasks on the Master/Worker nodes as defined in the Ansible hosts file.

* Configures the Docker Services for the HPE 3PAR Docker Volume Plug-in
* Configures iSCSI/FC/File
* Installs the HPE 3PAR Docker Volume Plug-in (Containerized version)
* For Kubernetes/OpenShift,
  * Deploys a Highly Available HPE etcd cluster used by the HPE 3PAR Volume Plug-in for Docker
* Deploys the HPE FlexVolume Driver

### Prerequisites:
  - Basic knowledge of Ansible and ssh keys
  - Install Ansible 2.5 or above as per [Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

  - Login to 3PAR via SSH to create entry in /\<user>\/.ssh/known_hosts file
  > **Note:** Entries for the Master and Worker nodes should already exist within the /\<user>\/.ssh/known_hosts file from the OpenShift installation. If not, you will need to log into each of the Master and Worker nodes as well to prevent connection errors from Ansible.

  - Clone the python-hpedockerplugin repository
    ```
    cd ~/workspace
    git clone https://github.com/hpe-storage/python-hpedockerplugin
    cd python-hpedockerplugin/ansible_3par_docker_plugin
    ```


  - Create the HPE 3PAR properties file `properties/plugin_configuration_properties.yml` based on your HPE 3PAR Storage array configuration. Some of the properties are mandatory and must be specified in the properties file while others are optional. You can see a sample properties file in the folder `properties/plugin_configuration_properties_sample.yml`
      ```
      vi properties/plugin_configuration_properties.yml
      ```
For more information on supported parameters:
https://github.com/hpe-storage/python-hpedockerplugin/tree/master/ansible_3par_docker_plugin

Copy this sample configuration
```yaml

INVENTORY:
  DEFAULT:
#Mandatory Parameters-----------------------------------------------------------------------------------

    # Specify the port to be used by HPE 3PAR plugin etcd cluster
    host_etcd_port_number: 23790
    # Plugin Driver - iSCSI
    hpedockerplugin_driver: hpedockerplugin.hpe.hpe_3par_iscsi.HPE3PARISCSIDriver
    hpe3par_ip: 10.90.97.1
    hpe3par_username: 3paradm
    hpe3par_password: 3pardata
    hpe3par_port: 8080
    hpe3par_cpg: NL_r6

    # Plugin version - Required only in DEFAULT backend
    volume_plugin: hpestorage/legacyvolumeplugin:3.3
    # Dory installer version - Required for Openshift/Kubernetes setup
    # Supported versions are dory_installer_v31, dory_installer_v32
    dory_installer_version: dory_installer_v32

#Optional Parameters------------------------------------------------------------------------------------

    logging: DEBUG
    hpe3par_snapcpg: NL_r6    

    # Uncomment to encrypt passwords in hpe.conf using defined passphrase
    #encryptor_key: < encrypt_key1 >

    #ssh_hosts_key_file: '/root/.ssh/known_hosts'
    #hpe3par_debug: True
    #suppress_requests_ssl_warning: True
    #hpe3par_iscsi_chap_enabled: True
    #use_multipath: False
    #enforce_multipath: False
    #vlan_tag: True
```

Save and exit

### Ansible hosts file
Modify the Ansible hosts file to define your Master/Worker nodes as well as where you want to deploy your etcd cluster.
>Note: We will be using the Ansible hosts file found in the ansible_3par_docker_plugin folder.

Make sure you are in the `python-hpedockerplugin/ansible_3par_docker_plugin` directory.
```
$ vi hosts
```

Change this to match your group.

```yaml
#Enable and populate the proxies here
#[all:vars]
#http_proxy=http://x.x.x.x:port_number
#https_proxy=https://x.x.x.x:port_number

[masters]
kube-g1-master1
kube-g1-master2
kube-g1-master3

[workers]
kube-g1-node1
kube-g1-node2

[etcd]
kube-g1-master1
kube-g1-master2
kube-g1-master3
```

Save and exit

Make sure you are in the `python-hpedockerplugin/ansible_3par_docker_plugin` directory.

```
ansible-playbook -i hosts install_hpe_3par_volume_driver.yml
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
