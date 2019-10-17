## Kubernetes 101

### The basics:

The first thing we need to do is to understand the various components of Kubernetes:

### Cluster
This is everything. All components that make up a Kubernetes deployment. This includes the control plane, master and worker nodes, and physical machines that allow you to run your container workloads on.

* **Control Plane:** The various parts of the Kubernetes Control Plane, such as the Kubernetes Master and kubelet processes, govern how Kubernetes communicates with your cluster. The Control Plane maintains a record of all of the Kubernetes Objects in the system, and runs continuous control loops to manage those objects’ state. At any given time, the Control Plane’s control loops will respond to changes in the cluster and work to make the actual state of all the objects in the system match the desired state that you provided.

> For example, when you use the Kubernetes API to create a Deployment, you provide a new desired state for the system. The Kubernetes Control Plane records that object creation, and carries out your instructions by starting the required applications and scheduling them to cluster nodes–thus making the cluster’s actual state match the desired state.

* **Kubernetes Master:** The Kubernetes master is responsible for maintaining the desired state for your cluster. When you interact with Kubernetes, such as by using the kubectl command-line interface, you’re communicating with your cluster’s Kubernetes master.

> The “master” refers to a collection of processes managing the cluster state. Typically all these processes run on a single node in the cluster, and this node is also referred to as the master. The master can also be replicated for availability and redundancy.

* **Kubernetes Nodes:** The nodes in a cluster are the machines (VMs, physical servers, etc) that run your applications and cloud workflows. The Kubernetes master controls each node; you’ll rarely interact with nodes directly.

### Kubernetes Objects

* **Pods:** A Pod is the basic execution unit of a Kubernetes application–the smallest and simplest unit in the Kubernetes object model that you create or deploy. A Pod encapsulates an application’s container (or, in some cases, multiple containers), storage resources, a unique network IP, and options that govern how the container(s) should run.

* **Deployments:** A Deployment provides declarative updates for Pods and ReplicaSets.

* **DaemonSet:** A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected.

* **Namespaces:** Kubernetes supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called namespaces. Namespaces are intended for use in environments with many users spread across multiple teams, or projects. Namespaces are a way to divide cluster resources between multiple users.

* **Services:** Expose an application running on a set of Pods as a network service.

* **Ingress:** Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster via Ingress Controllers. Traffic routing is controlled by rules defined on the Ingress resource.


**NEXT:** [Get to know your k8s cluster](get-to-know-cluster.md)
