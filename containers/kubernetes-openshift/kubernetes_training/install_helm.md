## Installing Helm

Helm helps you manage Kubernetes applications — Helm Charts help you define, install, and upgrade even the most complex Kubernetes application. There are two parts to Helm: The Helm client (helm) and the Helm server (Tiller). This guide shows how to install the client, and then proceeds to show two ways to install the server.

For more information, please refer to [https://helm.sh/docs/](https://helm.sh/docs/)

### INSTALLING THE HELM CLIENT
The Helm client can be installed either from source, or from pre-built binary releases.

### From the Binary Releases
Every release of Helm provides binary releases for a variety of OSes. These binary versions can be manually downloaded and installed.

1. Download your desired version
2. Unpack it (`tar -zxvf helm-v2.0.0-linux-amd64.tgz`)
3. Find the `helm` binary in the unpacked directory, and move it to its desired destination (`mv linux-amd64/helm /usr/local/bin/helm`)

From there, you should be able to run the client: `helm help`.

### From Chocolatey (Windows)
```
choco install kubernetes-helm
```

### INSTALLING TILLER
Tiller, the server portion of Helm, typically runs inside of your Kubernetes cluster.

###Easy In-Cluster Installation
The easiest way to install `tiller` into the cluster is simply to run:
```
helm init
```

This will validate that helm’s local environment is set up correctly (and set it up if necessary). Then it will connect to whatever cluster `kubectl` connects to by default (`kubectl config view`). Once it connects, it will install tiller into the kube-system namespace.

After helm init, you should be able to run `kubectl get pods --namespace kube-system` and see Tiller running.


**PREVIOUS:** [Installing Kubernetes Dashboard](dashboard.md)

**NEXT:** [Deploy Your First Pod](deploy_first_pod.md)
