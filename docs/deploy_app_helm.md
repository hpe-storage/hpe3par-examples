# Deploying Wordpress with Persistent volumes

Now that we have access to HPE Storage within Kubernetes, let's deploy an application with persistent volumes.

For this demo, we will be using WordPress as an example.

## Before you begin

There are some pre-requisites to ensure a successful deployment. If you have been following along and completing the tutorial, then you are ready to go, but let's review them nonetheless.

* Helm configured - Helm is used to automate the deployment of WordPress chart

* Default StorageClass - make sure you have a default StorageClass for use by the helm deployment
<br/>

  * look for the default flag
  ```
  $ kubectl get sc
  ```

  * refer to [Setting default StorageClass](default_storageclass.md)
<br/>
* Ingress controllers deployed for external access to your Kubernetes cluster **(pre-configured in HoL)**

  ```
  $ kubectl get ds -n kube-system | grep traefik
  ```
  * refer to [Configuring Ingress](optional_ingress.md)
<br/>
* Load Balancer configured and DNS to application **(pre-configured in HoL)**
  * refer to [Configuring Ingress](optional_ingress.md) for a simple **haproxy** example


## Install WordPress using Helm
We will be using the stable/WordPress chart from the upstream repo. First we need to install the `stable` repository:

```
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
$ helm repo updated
"stable" has been added to your repositories

$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
```

We can search the repo for available charts.
```
$ helm search repo wordpress
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
stable/wordpress        9.0.1           5.3.2           Web publishing platform for building blogs and ...
```

Now we can install WordPress.

We will customize it to our environment. We will specify the service and configure the ingress rules to point to our DNS name for the site.
```
$ helm install my-wordpress stable/wordpress --version 9.0.1 --set service.type=ClusterIP,ingress.enabled=true,ingress.hostname=wp.dev.<g#>.kubehol.net
```

It should look similar to this:
```
$ helm install my-wordpress stable/wordpress --version 9.0.1 --set service.type=ClusterIP,ingress.enabled=true,ingress.hostname=wp.dev.g18.kubehol.net
NAME: my-wordpress
LAST DEPLOYED: Mon Mar  2 23:33:46 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
** Please be patient while the chart is being deployed **

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL and associate WordPress hostname to your cluster external IP:

   export CLUSTER_IP=$(minikube ip) # On Minikube. Use: `kubectl cluster-info` on others K8s clusters
   echo "WordPress URL: http://wp.dev.g18.kubehol.net/"
   echo "$CLUSTER_IP  wp.dev.g18.kubehol.net" | sudo tee -a /etc/hosts

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: user
  echo Password: $(kubectl get secret --namespace default my-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)
```  

This will take a few minutes for everything to deploy. You can check the status with:
```
$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
my-wordpress-675b47f97-hnd8p       1/1     Running   0          77s
my-wordpress-mariadb-0             1/1     Running   0          77s
```


Finally, we can open a browser and go to:
```
http://wp.dev.g#.kubehol.net/
```

If everything was successful you will see your new WordPress site.

To get your username/password.
```deployment_name``` can be found by running ```helm list```.

```
  Login with the following credentials to see your blog

  echo Username: user
  echo Password: $(kubectl get secret --namespace default my-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)
```

You can log into the site and blog away!
```
http://wp.dev.g#.kubehol.net/admin
```


**HOME:** [Kubernetes Training Hands on Lab](https://hpe-storage.github.io/hpe3par-examples/)
