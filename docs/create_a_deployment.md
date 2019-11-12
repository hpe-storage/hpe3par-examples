# Exercise 6: Create a Deployment
The Kubernetes `Deployment` API object is the swiss-army workload type for single replica workloads using `ReadWriteOnce` (RWO) `PersistentVolumeClaim`s (PVC) but also capable of running multiple replicas with `ReadWriteMany` (RWX) `PVC`s.

In this example we'll use a hand-crafted `Deployment` specification with MariaDB.

# The Deployment mechanics
The `Deployment` object creates a `ReplicaSet` which in turn spawn `Pod`s. It's important to understand that some of the lifecycle controls provided with the `Deployment` makes it unsafe to run a `ReplicaSet` directly on `RWO` `PVC`s.

The stanza that matters for single replica `Deployments` using `RWO` `PVC`s look like this:
```
...
spec:
  replicas: 1
  strategy:
    type: Recreate
```

The default `.spec.strategy.type` is `RollingUpdate`, that translates to Kubernetes will do everything in its power to keep one replica alive, even it means running both at the same time. That will cause mount conflicts for `RWO` `PVC`s. Setting the key to `Recreate` ensures the previous `Pod` is properly terminated before creating a new when making changes to the `Deployment`.

## Create the workload

This workload contains a very simple MariaDB `Deployment` and a `PVC` that expects a default `StorageClass`:
```
kubectl create -f https://raw.githubusercontent.com/NimbleStorage/container-examples/master/misc/Prague19/k8s/deploy-mariadb.yaml
```

### Questions

* The `Deployment` is currently pulling down `:latest`, if you edit/patch the deployment to `:bionic`, does it come back up?
* If you delete the `Pod` of the `Deployment`, what do you observe?

# Learn more

Deployments on [kubernetes.io](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

* **PREVIOUS:** [Exercise 5: Create a StatefulSet](create_a_statefulset.md)
* **NEXT:** [Exercise 7: Set Namespace quotas](namespace_quotas.md)
