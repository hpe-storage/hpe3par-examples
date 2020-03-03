# Exercise 2: Install Kubernetes Dashboard

Dashboard is a web-based Kubernetes user interface. You can use Dashboard to deploy containerized applications to a Kubernetes cluster, troubleshoot your containerized application, and manage the cluster resources. You can use Dashboard to get an overview of applications running on your cluster, as well as for creating or modifying individual Kubernetes resources (such as Deployments, Jobs, DaemonSets, etc). For example, you can scale a Deployment, initiate a rolling update, restart a pod or deploy new applications using a deploy wizard.

Dashboard also provides information on the state of Kubernetes resources in your cluster and on any errors that may have occurred.

Please refer to [Kubernetes Web UI (Dashboard)](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

## Deploying the Dashboard UI
The Dashboard UI is not deployed by default. To deploy it, run the following command:

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
```

## Accessing the Dashboard UI
You can access Dashboard using `kubectl` from your desktop. Running this command will not work through SSH (i.e. Putty).
```
$ kubectl proxy
```

Open a web browser, copy the following URL to access the Dashboard.
```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

The Dashboard UI can only be accessed from the machine where the command is executed. See `kubectl proxy --help` for more options.


## Create the Admin Service Account

To protect your cluster data, Dashboard deploys with a minimal RBAC configuration by default. Currently, Dashboard only supports logging in with a Bearer Token. To create a token for this demo, we will create an admin user.

Warning: The admin user created in the tutorial will have administrative privileges and is for educational purposes only.

Open a second terminal, if you don't have one open already. Create the following file:

```
$ vi dashboard-adminuser.yml
```

Copy the code below into `dashboard-adminuser.yml` file
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
```

Run `kubectl apply`:

```
$ kubectl apply -f dashboard-adminuser.yml
```
---
>**Pro Tip** If you don't want to create a file everytime you want to create an object or deployment, you can run the **kubectl** command as follows:
>```
kubectl create -f-
>```
>Press **Enter** and you will be on a new line

>Copy and paste the code block:
>```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
>```

>Press **Enter**

>Finally press **Ctrl+D**

>The command will then be executed. You can use this method throughout these exercises.

---

### Create ClusterRoleBinding
Let's create the ClusterRoleBinding for the new admin-user. We will apply the `cluster-admin` role to the `admin-user`.

Copy the code below into `admin-role-binding.yml` file

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```  

Run `kubectl apply`:

```
$ kubectl apply -f dashboard-adminuser.yml
```

## Get Token

Now we are ready to get the token from the admin-user in order to log into the dashboard. Run the following command:

```
$ kubectl -n kube-system get secret | grep admin-user
```

It will return something similar to: `admin-user-token-n7jx9`

Now run:

```
$ kubectl -n kube-system describe secret admin-user-token-n7jx9
```

Copy the token value:

```yaml
Name:         admin-user-token-n7jx9
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: 7e9a4b56-e692-496a-8767-965076a282a4

Type:  kubernetes.io/service-account-token

Data
====
namespace:  11 bytes
token:      <your token will be shown here>
ca.crt:     1025 bytes
```

Switch back over to your browser and paste the token into the dashboard form then Click - Sign In.



Courtesy: [https://medium.com/@kanrangsan/creating-admin-user-to-access-kubernetes-dashboard-723d6c9764e4](https://medium.com/@kanrangsan/creating-admin-user-to-access-kubernetes-dashboard-723d6c9764e4)


**PREVIOUS:** [Exercise 1: Get to know your k8s cluster!](get_to_know_cluster.md)

**NEXT:** [Exercise 3: Deploy your first pod](deploy_first_pod.md)
