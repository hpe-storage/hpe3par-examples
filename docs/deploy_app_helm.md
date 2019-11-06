# Deploying Wordpress with Persistent volumes

Now that we have access to HPE Storage within Kubernetes, let's deploy an application with persistent volumes.

For this demo, we will be using WordPress as a good example.

## Before you begin

There are some pre-requisites to ensure a successful deployment. If you have been following along and completing the tutorial, then you are ready to go, but let's review them nonetheless.

* Helm & Tiller configured - Helm is used to automate the deployment of WordPress chart
* Default StorageClass - make sure you have a default StorageClass for use by the helm deployment
  * `kubectl get sc` look for default flag
  * refer to [Setting default StorageClass](default_storageclass.md)
* Ingress controllers deployed for external access to your Kubernetes cluster **(pre-configured in HoL)**
  * `kubectl get all -n kube-system | grep traefik`
  * refer to [Configuring Ingress](optional_ingress.md)
* Load Balancer configured and DNS to application **(pre-configured in HoL)**
  * refer to [Configuring Ingress](optional_ingress.md) for a simple **haproxy** example


 ## Add Repo

 We will be using the WordPress chart from the bitnami repo. The following command allows you to download and install all the charts from this repository, both the bitnami and the upstreamed ones.
 ```
 $ helm repo add bitnami https://charts.bitnami.com
 ```

Once we have added the repo, we can search it for available charts.
```
$ helm search wordpress
```

Now we can install WordPress. The base command to install is:
```
$ helm install stable/wordpress
```

but we want to customize it to our environment. We will specify the service type and configure the ingress rules.
```
helm install stable/wordpress --set serviceType=ClusterIP,ingress.enabled=true,ingress.hostname=wp.dev.g<group_number>.example.com
```

It should look similar to this:
```
helm install stable/wordpress --set serviceType=ClusterIP,ingress.enabled=true,ingress.hostname=wp.dev.g10.example.com
```

This will take a few minutes to complete.

Once complete, open a browser and go to:
```
http://wp.dev.g<group_number>.example.com
```

It should look like the following:
```
http://wp.dev.g10.example.com
```

If everything was successful you will see your new WordPress site.


**PREVIOUS:** [Deploying app with Helm](deploy_app_helm.sh)

**NEXT:**
