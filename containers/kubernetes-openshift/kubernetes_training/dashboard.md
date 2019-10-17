### Accessing the Dashboard UI:

### Command line proxy
You can access Dashboard using the kubectl command-line tool by running the following command:
```
kubectl proxy
```
Kubectl will make Dashboard available at:
```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

The UI can only be accessed from the machine where the command is executed. See `kubectl proxy --help` for more options.

To protect your cluster data, Dashboard deploys with a minimal RBAC configuration by default. Currently, Dashboard only supports logging in with a Bearer Token. To create a token for this demo, we will create an admin user.

Open a second terminal, if you don't have one open already.

### Create the Admin Service Account

>Warning: The admin user created in the tutorial will have administrative privileges and is for educational purposes only.

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
# kubectl apply -f dashboard-adminuser.yml
```

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
# kubectl apply -f dashboard-adminuser.yml
```

### Get Token

Now we are ready to get the token from the admin-user in order to log into the dashboard. Run the following command:

```
kubectl -n kube-system get secret | select-string -pattern admin-user
```

It will return something similar to:

```
admin-user-token-n7jx9
```

Now run:

```
kubectl -n kube-system describe secret admin-user-token-n7jx9
```

Copy the token value:

```
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

Paste it into the dashboard form then Click - Sign In.

Courtesy: [https://medium.com/@kanrangsan/creating-admin-user-to-access-kubernetes-dashboard-723d6c9764e4](https://medium.com/@kanrangsan/creating-admin-user-to-access-kubernetes-dashboard-723d6c9764e4)


**PREVIOUS:** [Get to know your k8s cluster!](get_to_know_cluster.md)

**NEXT:** [Installing Helm](install_helm.md)
