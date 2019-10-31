# Why change the default storage class?

https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/

Depending on the installation method, your Kubernetes cluster may or may not be deployed with an existing StorageClass that is marked as default. This default StorageClass is then used to dynamically provision storage for `PersistentVolumeClaims` that do not require any specific storage class. See [PersistentVolumeClaim documentation](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#class-1) for details.

The pre-installed default StorageClass may not fit well with your expected workload; for example, it might provision storage that is too expensive. If this is the case, you can either change the default `StorageClass` or disable it completely to avoid dynamic provisioning of storage.

## Changing the default StorageClass

List the StorageClasses in your cluster:
```
  kubectl get storageclass
```

The output is similar to this:
```
  NAME                 PROVISIONER               AGE
  standard (default)   kubernetes.io/gce-pd      1d
  gold                 kubernetes.io/gce-pd      1d
```

The default StorageClass is marked by `(default)`.


#### Mark the default StorageClass as non-default:

The default StorageClass has an annotation `storageclass.kubernetes.io/is-default-class` set to `true`. Any other value or absence of the annotation is interpreted as `false`.

To mark a StorageClass as non-default, you need to change its value to false:
```
  kubectl patch storageclass <your-class-name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
```

where `<your-class-name>` is the name of your chosen StorageClass.

#### Mark a StorageClass as default:

Similarly to the previous step, you need to add/set the annotation **storageclass.kubernetes.io/is-default-class=true**.

```
  kubectl patch storageclass <your-class-name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

Please note that at most one StorageClass can be marked as default. If two or more of them are marked as default, a `PersistentVolumeClaim` without `storageClassName` explicitly specified cannot be created.

#### Verify that your chosen StorageClass is default:
```
  kubectl get storageclass
```

The output is similar to this:
```
  NAME             PROVISIONER               AGE
  standard         kubernetes.io/gce-pd      1d
  gold (default)   kubernetes.io/gce-pd      1d
```  
