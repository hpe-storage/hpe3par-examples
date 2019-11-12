# Exercise 5: Create a StatefulSet
A `StatefulSet` is something that derived from something that was called a `PetSet` prior to Kubernetes 1.5. The name implies that some attributes of the workload need to persist and be taken care of.

StatefulSets provide ordered starts and stable naming for each replica and each replica may each also consume a `PersistentVolumeClaim` (PVC) with a predictable name. This introduces the `volumeClaimTemplates` stanza in the `spec`:

```
...
  volumeClaimTemplates:
  - metadata:
      name: vol
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 32Gi
```

This stanza which is part of [this declaration](https://raw.githubusercontent.com/NimbleStorage/container-examples/master/blogs/flexvolume-3.0/install/statefulset-iozone.yaml) will simply create a 32GiB volumes from the default `StorageClass` on the cluster for each replica. It's also possible to create `PVC`s from a custom `StorageClass` using the `storageClassName`, just like in a standard `PVC`.

Let's deploy and inspect.

```
kubectl create -f https://raw.githubusercontent.com/NimbleStorage/container-examples/master/blogs/flexvolume-3.0/install/statefulset-iozone.yaml
```

### Questions

* What does `kubectl get pvc -w` look like? (be patient)
* If you `kubectl exec` into one of the `Pods`, what is the hostname?
* If you `kubectl scale --replicas=5 statefulset/io`, what can you observe on the cluster?

**Hint:** Deleting the `StatefulSet` won't delete `PVC`s.

# Learn more

StatefulSets on [kubernetes.io](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)

* **PREVIOUS:** [Exercise 4: Customized StorageClass and PVC](customize_storageclass.md)
* **NEXT:** [Exercise 6: Create a Deployment](create_a_deployment.md)
