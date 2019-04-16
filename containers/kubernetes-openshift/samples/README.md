# Example Storage Class, PVC, and Pod using the HPE 3PAR Docker Volume plugin

Pre-requisite:
* Kubernetes deployed
* HPE 3PAR Docker volume plugin configured
* Copy the examples in this repo

Starting with the Storage StorageClass

>Note: the commands below are ran using OpenShift commands, they can swapped with kubectl if running Kubernetes

```
oc create -f sc_example.yml

```

You can view the details of the StorageClass by running the following command:
```
oc describe sc sc-gold

```

Let's create a PVC
```
oc create -f pvc_example.yml

```

You can view the details of the PVC and check to see when it is Bound
```
$ oc describe pvc pvc-nginx

Name:          pvc-nginx
Namespace:     default
StorageClass:  sc-gold
Status:        Bound
Volume:        sc-gold-654505f0-2ef3-11e9-939e-005056a22f34
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed=yes
               pv.kubernetes.io/bound-by-controller=yes
               volume.beta.kubernetes.io/storage-provisioner=hpe.com/hpe
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      10Gi
Access Modes:  RWO
Events:        <none>

```

Now lets create a pod.

```

oc create -f nginx-pod.yml

```

Wait a few minutes for the pod to create and the volume to mount.

You can watch it's status with:
```
oc describe pod pod-nginx
```

Now let's log into the pod and write some data.

```
kubectl exec -it pod-nginx -- /bin/bash
```

```
echo Hello from Kubernetes Storage! > /usr/share/nginx/html/index.html
```

Go ahead and log out of the pod.

Now lets check to see if everything is working properly.

```
$ oc describe pod pod-nginx | less
Name:               pod-nginx
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               o-node1.virtware.io/10.10.1.101
Start Time:         Tue, 16 Apr 2019 14:02:54 -0600
Labels:             name=nginx-hpe
Annotations:        openshift.io/scc=anyuid
Status:             Running
IP:                 10.130.0.3

```
Look through the oc describe until you find the IP address for the pod. We will need the IP to verify everything is working.

```
$ curl 10.130.0.3:80

Hello from Kubernetes Storage!
```

If you see the output above, you have successfully create a working persistent volume, claim and pod.
