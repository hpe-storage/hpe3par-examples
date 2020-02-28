# Exercise 4: Installing Helm

Helm helps you manage Kubernetes applications — Helm Charts help you define, install, and upgrade even the most complex Kubernetes application. There are two parts to Helm: The Helm client (helm) and the Helm server (Tiller). This guide shows how to install the client, and then proceeds to show two ways to install the server.

For more information, please refer to [https://helm.sh/docs/](https://helm.sh/docs/)

## INSTALLING THE HELM CLIENT
The Helm client can be installed either from source, or from pre-built binary releases.

## From the Binary Releases
Every release of Helm provides binary releases for a variety of OSes. These binary versions can be manually downloaded and installed.

1. Download your desired version. https://github.com/helm/helm/releases/latest
2. Unpack it (`tar -zxvf helm-v3*.tgz`)
3. Find the `helm` binary in the unpacked directory, and move it to its desired destination (`mv linux-amd64/helm /usr/local/bin/helm`)

From there, you should be able to run the client: `helm help`.

## From Chocolatey (Windows)
```
choco install kubernetes-helm
```

Once you have Helm ready, you can add a chart repository. One popular starting location is the official Helm stable charts:
```
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

Once this is installed, you will be able to list the charts you can install:

```helm search repo stable
NAME                                    CHART VERSION   APP VERSION                     DESCRIPTION
stable/acs-engine-autoscaler            2.2.2           2.1.1                           DEPRECATED Scales worker nodes within agent pools
stable/aerospike                        0.2.8           v4.5.0.5                        A Helm chart for Aerospike in Kubernetes
stable/airflow                          4.1.0           1.10.4                          Airflow is a platform to programmatically autho...
stable/ambassador                       4.1.0           0.81.0                          A Helm chart for Datawire Ambassador
# ... and many more
```

**PREVIOUS:** [Exercise 3: Deploy your first pod](deploy_first_pod.md)

**NEXT:** [Exercise 5: Deploy HPE 3PAR/Primera CSI Driver](3par_volume_plugin_install.md) or [Deploy HPE Nimble CSI Driver](nimble_volume_plugin_install.md)
