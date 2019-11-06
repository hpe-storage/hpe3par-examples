# Deploying Wordpress with Persistent volumes

Now that we have access to HPE Storage within Kubernetes, let's deploy an application with persistent volumes.

For this demo, we will be using WordPress as a good example.

## Before you begin

There are some pre-requisites to ensure a successful deployment. If you have been following along and completing the tutorial, then you are ready to go, but let's review them nonetheless.

* Helm & Tiller configured - Helm is used to automate the deployment of WordPress chart
* Default StorageClass - make sure you have a default StorageClass for use by the helm deployment

  * ```kubectl get sc ``` look for the default flag
  * refer to [Setting default StorageClass](default_storageclass.md)

* Ingress controllers deployed for external access to your Kubernetes cluster **(pre-configured in HoL)**
  * ```kubectl get ds -n kube-system | grep traefik```
  * refer to [Configuring Ingress](optional_ingress.md)
* Load Balancer configured and DNS to application **(pre-configured in HoL)**
  * refer to [Configuring Ingress](optional_ingress.md) for a simple **haproxy** example


## Install WordPress using Helm
We will be using the stable/WordPress chart from the upstream repo. We can search the repo for available charts.
```
$ helm search wordpress
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
stable/wordpress        7.6.0           5.2.4           Web publishing platform for building blogs and websites.
```

Now we can install WordPress using the defaults but that isn't what we want.
```
$ helm install stable/wordpress
```

We will  customize it to our environment. We will specify the service and configure the ingress rules to point to our DNS name for the site.
```
$ helm install stable/wordpress --set serviceType=ClusterIP,ingress.enabled=true,ingress.hostname=wp.dev.g<group_number>.example.com
```

It should look similar to this:
```
PS C:\Users\Administrator> helm install stable/wordpress --set serviceType=ClusterIP,ingress.enabled=true,ingress.hostname=wp.dev.g10.example.com
NAME:   snug-turtle
LAST DEPLOYED: Mon Nov  4 20:20:43 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                       DATA  AGE
snug-turtle-mariadb        1     0s
snug-turtle-mariadb-tests  1     0s

==> v1/Deployment
NAME                   READY  UP-TO-DATE  AVAILABLE  AGE
snug-turtle-wordpress  0/1    1           0          0s

==> v1/PersistentVolumeClaim
NAME                   STATUS   VOLUME        CAPACITY  ACCESS MODES  STORAGECLASS  AGE
snug-turtle-wordpress  Pending  hpe-standard  0s

==> v1/Pod(related)
NAME                                   READY  STATUS   RESTARTS  AGE
snug-turtle-mariadb-0                  0/1    Pending  0         0s
snug-turtle-wordpress-57885c775-ds97z  0/1    Pending  0         0s

==> v1/Secret
NAME                   TYPE    DATA  AGE
snug-turtle-mariadb    Opaque  2     0s
snug-turtle-wordpress  Opaque  1     0s

==> v1/Service
NAME                   TYPE          CLUSTER-IP     EXTERNAL-IP  PORT(S)                     AGE
snug-turtle-mariadb    ClusterIP     10.233.57.67   <none>       3306/TCP                    0s
snug-turtle-wordpress  LoadBalancer  10.233.62.107  <pending>    80:30767/TCP,443:30869/TCP  0s

==> v1/StatefulSet
NAME                 READY  AGE
snug-turtle-mariadb  0/1    0s

==> v1beta1/Ingress
NAME                   AGE
snug-turtle-wordpress  0s


NOTES:
1. Get the WordPress URL:

  You should be able to access your new WordPress installation through

2. Login with the following credentials to see your blog

  echo Username: user
  echo Password: $(kubectl get secret --namespace default snug-turtle-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)
```  

This will take a few minutes for everything to deploy. You can check the status with:
```
$ kubectl get pods
NAME                                    READY   STATUS              RESTARTS   AGE
snug-turtle-mariadb-0                   1/1     Running             0          2m37s
snug-turtle-wordpress-57885c775-ds97z   1/1     Running             0          2m37s
```

Once complete, open a browser and go to:
```
http://wp.dev.g<group_number>.example.com
```

It should look like the following:
```
http://wp.dev.g10.example.com
```



If everything was successful you will see your new WordPress site.

Remember to get your username/password. ```deployment_name``` can be found by running ```helm list```.
```
  Login with the following credentials to see your blog

  echo Username: user
  echo Password: $(kubectl get secret --namespace default <deployment_name> -o jsonpath="{.data.wordpress-password}" | base64 --decode)
```

You can log into the site:
```
http://wp.dev.g<group_number>.example.com/admin
```




**PREVIOUS:** [Deploying app with Helm](deploy_app_helm.sh)
