# Exercise 7: Set Namespace quotas
There are a few storage related `ResourceQuotas` that may be set on the namespace where the `PersistentVolumeClaim` (PVC) is created. These are practical to set for sprawly users and workloads that have no concept of object count or capacity limits.

## Limit capacity
```
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: capacity
spec:
  hard:
    requests.storage: 1Gi
```

Inspect the capacity quota:
```
kubectl get quota -o yaml
```

This exercise is not comprehensive. Please see "Learn more" below for more examples.

### Questions

* What do you think would happen if you create another `PVC`?

# Learn more

Storage Resource Quota on [kubernetes.io](https://kubernetes.io/docs/concepts/policy/resource-quotas/#storage-resource-quota)

* **PREVIOUS:** [Exercise 6: Create a Deployment](create_a_deployment.md)
* **NEXT:** [Bonus content: Tips & Tricks](tips_and_tricks.md)
