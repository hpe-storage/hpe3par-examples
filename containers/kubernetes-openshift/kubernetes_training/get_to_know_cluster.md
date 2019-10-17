## Kubernetes 101

All of this information is taken from the official documentation found on  [kubernetes.io/docs](https://kubernetes.io/docs/)

### Overview of kubectl

The Kubernetes command-line tool, kubectl, allows you to run commands against Kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs. For a complete list of kubectl operations, see [Overview of kubectl](https://kubernetes.io/docs/reference/kubectl/overview/).

For more information on how to install and setup `kubectl` on Linux, Windows or MacOS, see [Install and Set Up kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

### Syntax
Use the following syntax to run kubectl commands from your terminal window:

`kubectl [command] [TYPE] [NAME] [flags]`

where `command`, `TYPE`, `NAME`, and `flags` are:

* `command`: Specifies the operation that you want to perform on one or more resources, for example create, get, describe, delete.

* `TYPE`: Specifies the resource type. Resource types are case-insensitive and you can specify the singular, plural, or abbreviated forms. For example, the following commands produce the same output:
```
  kubectl get pod pod1
  kubectl get pods pod1
  kubectl get po pod1
```

* `NAME`: Specifies the name of the resource. Names are case-sensitive. If the name is omitted, details for all resources are displayed, for example `kubectl get pods`.

[Kubernetes Cheat Sheet](Kubernetes-Cheat-Sheet.pdf) - Courtesy of Linux Academy


### Getting to know your cluster:

Let's run through some simple kubectl commands to get familiar with your cluster.

In order to communicate with the Kubernetes cluster, `kubectl` looks for a file named config in the `$HOME/.kube` directory. You can specify other `kubeconfig` files by setting the KUBECONFIG environment variable or by setting the `--kubeconfig` flag.

To view your config file:
```
kubectl config view
```


Check that kubectl and the config file is properly configured by getting the cluster state:

```
kubectl cluster-info
```
If you see a URL response, kubectl is correctly configured to access your cluster.

```
# kubectl cluster-info
Kubernetes master is running at https://10.90.200.11:6443
coredns is running at https://10.90.200.11:6443/api/v1/namespaces/kube-system/services/coredns:dns/proxy
kubernetes-dashboard is running at https://10.90.200.11:6443/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

Now lets look at the nodes within our cluster.
```
kubectl get nodes
```
You should see output similar to below. As you can see, each node has a role as master or worker node (\<none>).
```
# kubectl get nodes
NAME              STATUS   ROLES    AGE   VERSION
kube-g1-master1   Ready    master   37d   v1.15.3
kube-g1-master2   Ready    master   37d   v1.15.3
kube-g1-master3   Ready    master   37d   v1.15.3
kube-g1-node1     Ready    <none>   37d   v1.15.3
kube-g1-node2     Ready    <none>   37d   v1.15.3
```

You can list any available pods.
```
kubectl get pods
```


**PREVIOUS:** [Kubernetes 101](kubernetes101.md)

**NEXT:** [Installing Kubernetes Dashboard](dashboard.md)
