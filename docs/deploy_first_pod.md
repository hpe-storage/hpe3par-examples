# Exercise 3: Deploy your first pod


A pod is a collection of containers sharing a network and mount namespace and is the basic unit of deployment in Kubernetes. All containers in a pod are scheduled on the same node.

Lets create a simple nginx webserver.

```
$ vi first-nginx-pod.yml
```

Copy and paste the following
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: first-nginx-pod
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx-first-pod
  template:
    metadata:
      labels:
        run: nginx-first-pod
    spec:
      containers:
      - image: nginx
        name: nginx
```        
Save and exit

Create the pod, execute:
```
$ kubectl apply -f first-nginx-pod.yml
```

We can now see the pod running:

```
$ kubectl get pods

NAME                               READY   STATUS    RESTARTS   AGE
first-nginx-pod-5bb4787f8d-7ndj6   1/1     Running   0          6m39s
```

We can inspect the pod further using the **kubectl describe** command:
```
Name:         first-nginx-pod-5bb4787f8d-7ndj6
Namespace:    default
Priority:     0
Node:         kube-g18-node1/10.90.200.184
Start Time:   Mon, 02 Mar 2020 17:09:20 -0600
Labels:       pod-template-hash=5bb4787f8d
              run=nginx-first-pod
Annotations:  <none>
Status:       Running
IP:           10.233.82.7
IPs:
  IP:           10.233.82.7
Controlled By:  ReplicaSet/first-nginx-pod-5bb4787f8d
Containers:
  nginx:
    Container ID:   docker://a0938f10d28cb0395b0c2c324e29236f0c74ecdcdc63e556863c53ee7a88d56d
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:380eb808e2a3b0dd954f92c1cae2f845e6558a15037efefcabc5b4e03d666d03
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 02 Mar 2020 17:09:32 -0600
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-m2vbl (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-m2vbl:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-m2vbl
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From                     Message
  ----    ------     ----       ----                     -------
  Normal  Scheduled  <unknown>  default-scheduler        Successfully assigned default/first-nginx-pod-5bb4787f8d-7ndj6 to kube-g18-node1
  Normal  Pulling    54s        kubelet, kube-g18-node1  Pulling image "nginx"
  Normal  Pulled     46s        kubelet, kube-g18-node1  Successfully pulled image "nginx"
  Normal  Created    44s        kubelet, kube-g18-node1  Created container nginx
  Normal  Started    43s        kubelet, kube-g18-node1  Started container nginx
```

Lets find the IP address of the pod.
```
$ kubectl describe pod <pod_name> | grep IP:
```

```
$ kubectl describe pod first-nginx-pod-5bb4787f8d-7ndj6 | grep IP:
IP:           10.233.82.7
  IP:           10.233.82.7

```

This IP address (10.233.82.7) is only accessible from within the cluster, so lets use `port-forward` to expose the `pod` port temporarily outside the cluster.

```
$ kubectl port-forward first-nginx-pod-5bb4787f8d-7ndj6 5000:80

Forwarding from 127.0.0.1:5000 -> 80
Forwarding from [::1]:5000 -> 80
```
>Note: port-forward is meant for temporarily exposing an application outside of a Kubernetes cluster. For a more permanent solution, look into Ingress.

Finally, we can open a browser and go to:
```
http://127.0.0.1:5000/
```

If you see, **Welcome to nginx!**, you have successfully deployed your first pod.

Now let's log into our pod. If you don't already, open another shell and run:

```
$ kubectl exec -it first-nginx-pod-5bb4787f8d-7ndj6 /bin/bash
```

You can explore the pod and run commands:

```
root@first-nginx-pod-5bb4787f8d-7ndj6:/# df -h
Filesystem               Size  Used Avail Use% Mounted on
overlay                   46G  8.0G   38G  18% /
tmpfs                     64M     0   64M   0% /dev
tmpfs                    1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root   46G  8.0G   38G  18% /etc/hosts
shm                       64M     0   64M   0% /dev/shm
tmpfs                    1.9G   12K  1.9G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                    1.9G     0  1.9G   0% /proc/acpi
tmpfs                    1.9G     0  1.9G   0% /proc/scsi
tmpfs                    1.9G     0  1.9G   0% /sys/firmware

```

Or modify the webpage:

```
$ vi /usr/share/nginx/html/index.html
```

Once done, exit the pod.


**PREVIOUS:** [Exercise 2: Install Kubernetes Dashboard](dashboard.md)

**NEXT:** [Exercise 4: Deploy HPE 3PAR/Primera CSI Driver](3par_volume_plugin_install.md) or [Deploy HPE Nimble CSI Driver](nimble_volume_plugin_install.md)
