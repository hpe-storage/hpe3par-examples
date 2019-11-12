# Tips & Tricks with Kubernetes
This page contains tricks on how to use your lab enironment more effectively.

## Copy & Paste YAML into a Terminal
Sometimes it's practical to paste a chunk directly into a terminal with copying or concatening files in between.

### Windows
In PowerShell:

```
PS C:\Users\Administrator\tmp> kubectl.exe create -f- <Hit ENTER and paste content>
---
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: database
provisioner: hpe.com/nimble
parameters:
  allowOverrides: limitIOPS
  limitIOPS: "10000"
  perfPolicy: "SQL Server"
reclaimPolicy: Delete
^Z <Hit CTRL+Z and hit ENTER on a new line>
storageclass.storage.k8s.io/database created
```

### Linux
In a bash shell:

```
$ kubectl create -f- <Hit ENTER and paste content>
---
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: database
provisioner: hpe.com/nimble
parameters:
  allowOverrides: limitIOPS
  limitIOPS: "10000"
  perfPolicy: "SQL Server"
reclaimPolicy: Delete
<Hit CTRL+D on a new line>
storageclass.storage.k8s.io/database created
```
