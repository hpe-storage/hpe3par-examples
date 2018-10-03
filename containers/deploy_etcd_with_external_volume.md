# How to deploy etcd with an external volume

### Use Case:

In our standard [install guide](https://github.com/budhac/python-hpedockerplugin/blob/master/docs/quick_start_guide.md), we deploy **etcd** for the HPE 3PAR Docker Volume plugin. The data within **etcd** is critical for the health and stability of your Docker deployments along with 3PAR volumes as well as the recovery of your Docker environment in the case of disaster. **Etcd** contains all of the metadata for the 3PAR volumes in use within your Docker/Kubernetes/OpenShift environments. This guide will show you how to protect the data in **etcd** by moving it outside of the container onto a 3PAR volume thus allowing you to protect **etcd**.

### Assumptions:
* You have existing data protection policies on your 3PAR volumes used in your Docker environment
* The **etcd** hosts (on management nodes or external to the environment) have data paths (iSCSI or FC) to the 3PAR array
* FC or iSCSI is pre-configured on your **etcd** hosts

Currently, in the [install guide](https://github.com/budhac/python-hpedockerplugin/blob/master/docs/quick_start_guide.md), we install **etcd** in the following manner:

#### etcd without external storage

```
sudo docker run -d -v /usr/share/ca-certificates/:/etc/ssl/certs -p 4001:4001 \
-p 2380:2380 -p 2379:2379 \
--name etcd quay.io/coreos/etcd:v2.2.0 \
-name etcd0 \
-advertise-client-urls http://${HostIP}:2379,http://${HostIP}:4001 \
-listen-client-urls http://0.0.0.0:2379,http://0.0.0.0:4001 \
-initial-advertise-peer-urls http://${HostIP}:23800 \
-listen-peer-urls http://0.0.0.0:2380 \
-initial-cluster-token etcd-cluster-1 \
-initial-cluster etcd0=http://${HostIP}:2380 \
-initial-cluster-state new
```

If you notice, there are no data volumes (**-v /host_directory:/container_directory**) attached to this container. This means that the **etcd** cluster data (including all of the 3PAR metadata, volume mappings to containers) exists only within the containers and as long as the etcd cluster stays healthy, the Docker environment and plugin along with your Docker volumes will be happy.

Anyone with much IT experience knows that accidents or outages happen, both planned and unplanned. Without a full backup of the **etcd** hosts and your Docker environment, there is a chance data loss and increased outage windows. And the last thing you want is to lose all of the configuration data for your Docker environment.

Again, we are assuming you have protection policies in place for your 3PAR Docker volumes and Docker container states. If you don't want to do full backup restores of your **etcd** hosts (which can be very time consuming especially if they are bare-metal), then we need to ensure that we are protecting the **etcd** data. The best way to do that is by redirecting the **etcd** data directories to an external volume preferably on a 3PAR array so that you can apply the same data protection policies already in place within your environment onto your **etcd** data volumes.

1. First things first, create and export an **etcd** volume (i.e. etcd1_vol) to your **etcd** hosts.

>**Note:** It is best practice, when deploying etcd in a cluster, that you create separate volumes (**etcd_node1_vol**, **etcd_node2_vol**, **etcd_node3_vol**, etc.) for each node in the cluster in order to maintain maximum data resiliency.

2. Mount the volume (etcd1_vol) to the etcd host OS.

```
$ lsblk -f
NAME                   FSTYPE       LABEL           UUID                                   MOUNTPOINT
sdb                    mpath_member                 2ce2ce3c-899c-480d-aed2-3bceddc32efa
└─mpathe               ext4                         2ce2ce3c-899c-480d-aed2-3bceddc32efa   /etcd_data
sdc                    mpath_member                 2ce2ce3c-899c-480d-aed2-3bceddc32efa
└─mpathe               ext4                         2ce2ce3c-899c-480d-aed2-3bceddc32efa   /etcd_data
```

3. Ensure that hpe.conf exists and is configured correctly per [install guide](https://github.com/budhac/python-hpedockerplugin/blob/master/docs/quick_start_guide.md).

#### etcd with external storage

```
sudo docker run -d -v /usr/share/ca-certificates/:/etc/ssl/certs -v /etcd_data:/etcd-data -p 4001:4001 \
-p 2380:2380 -p 2379:2379 \
--name etcd quay.io/coreos/etcd:v2.2.0 \
-name etcd0 -data-dir=/etcd-data  \
-advertise-client-urls http://${HostIP}:2379,http://${HostIP}:4001 \
-listen-client-urls http://0.0.0.0:2379,http://0.0.0.0:4001 \
-initial-advertise-peer-urls http://${HostIP}:2380 \
-listen-peer-urls http://0.0.0.0:2380 \
-initial-cluster-token etcd-cluster-1 \
-initial-cluster etcd0=http://${HostIP}:2380 \
-initial-cluster-state new
```

There are two important flags here.

* **-v /etcd_data:/etcd-data**    
  * **/etcd_data** is a 3PAR volume mounted to the local Linux filesystem
* **-data-dir=/etcd-data**    
  * **-data-dir** specifies where etcd is to write data

4. Install the HPE 3PAR Docker volume plugin

By specifying an external 3PAR volume the data is securely written directly to the 3PAR where you can backup/snapshot/clone the volume to ensure that it is protected in the case of a disaster.

It is important to know that as long as the etcd data is protected, in the case of a disaster, the **etcd** volume can be remounted to a new **etcd** host, and then re-run the **etcd with external storage** command. Upon reinstalling the HPE 3PAR Docker Volume plugin, the plugin will rescan **etcd** and see the existing volumes and container data allowing you to reestablish the links your containers and volumes back to working state.
