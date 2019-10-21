## Kubernetes Ingress

### What is Ingress?
Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.
```
    internet
        |
   [ Ingress ]
   --|-----|--
   [ Services ]
  -|----|-----|-
  Pod  Pod   Pod
```   
An Ingress can be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name based virtual hosting. An Ingress controller is responsible for fulfilling the Ingress, usually with a load balancer, though it may also configure your edge router or additional frontends to help handle the traffic.

### Prerequisites
* You must have an ingress controller to satisfy an Ingress.
* Configure a load balancer for your environment.
* DNS configured for wildcard support

Very good video on the subject: https://www.youtube.com/watch?v=A_PjjCM1eLA

### Setup Load Balancer

We won't go in depth on this since there are many ways to setup a load balancer and it will vary depending on the type of load balancer used. In this demo, we will use a simple `haproxy` load balancer in front of our worker nodes to route traffic into our Kubernetes cluster. You can run `haproxy` as a container or within a VM.

Here is a sample `http` load balancer config `/etc/haproxy/haproxy.cfg`:

```
frontend http_front
  bind *:80
  stats uri /haproxy?stats
  default_backend http_back


backend http_back
  balance roundrobin
  server kube1 192.168.1.176:80
  server kube2 192.168.1.177:80
```
Enable and start the haproxy service.  

### Setup Ingress Controllers

In order to route traffic internally within the Kubernetes clusters, we will need to deploy Ingress controllers.
We will be using Traefik to handle this. Please check out this page for more information: https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/

#### Deploy Traefik 1.7:
>If you run into any issues when following these steps, double check the latest version of Traefik. You may need to update the command accordingly but as of this writing, the latest is 1.7.

Create the Traefik Service Account and RBAC rules:
```
# kubectl apply -f https://raw.githubusercontent.com/containous/traefik/v1.7/examples/k8s/traefik-rbac.yaml
```

Inspect the clusterrole binding
```
# kubectl describe clusterrole traefik-ingress-controller -n kube-system
```

We will be deploying the Traefik Ingress Controllers as a DaemonSet which is good for a small number of worker nodes. If you are deploying to larger clusters, you may want to use a deployment to reduce the number of controllers deployed.

Create `traefik-controller-ds.yml`:

```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
---
kind: DaemonSet
apiVersion: extensions/v1beta1
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
spec:
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      containers:
      - image: traefik:v1.7
        name: traefik-ingress-lb
        ports:
        - name: http
          containerPort: 80
          hostPort: 80
        - name: admin
          containerPort: 8080
        securityContext:
          capabilities:
            drop:
            - ALL
            add:
            - NET_BIND_SERVICE
        args:
        - --api
        - --kubernetes
        - --logLevel=INFO
---
kind: Service
apiVersion: v1
metadata:
  name: traefik-ingress-service
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
    - protocol: TCP
      port: 80
      name: web
    - protocol: TCP
      port: 8080
      name: admin
```

#### Deploy the Traefik controllers:
```
# kubectl apply -f traefik-controller-ds.yml
```

Inspect the controllers
```
# kubectl get all -n kube-system | grep traefik
```

Now we are ready to create Services and Ingress Resources.

We will use a previous example from our first Pod deployment:

Create `first-nginx-pod.yml`

Copy and paste the following
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: first-nginx-pod
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx-first-pod
  template:
    metadata:
      labels:
        run: nginx-first-pod
    spec:
      containers:
      - image: nginx
        name: nginx
```   

Save and Exit
```
kubectl create -f first-nginx-pod.yaml
```

#### Create the Service
>**ClusterIP vs NodePort:**
>`ClusterIP` assigns each deployment/pod an internal cluster IP only routable within the cluster. This allows multiple services (http, databases, etc.) to use the same Port regardless of the worker node the pods are running on. Choosing `ClusterIP` makes the service only reachable from within the cluster. This is the default ServiceType.
`NodePort` assigns the pod to a fixed port on the node.  You’ll be able to contact the `NodePort` service, from outside the cluster, by requesting `<NodeIP>:<NodePort>`.

```
kubectl expose deploy first-nginx-pod --port 80
```

Check the created Service

```
# kubectl get service
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes           ClusterIP   10.233.0.1      <none>        443/TCP   6d23h
first-nginx-pod      ClusterIP   10.233.39.195   <none>        80/TCP    48m
```

#### Create the Ingress Resource
Finally lets create the Ingress Resource or rules that tie all of this together. Now that we have a load balancer routing traffic, ingress controllers to help route traffic to the pods on each worker node, and services in place to expose the application port, lets create the rules that tell the ingress controllers where to forward the traffic requests from the load balancer and map it to the appropriate Service so it reaches the applications.

Create `ingress-resource.yml`
>You may need to modify the host to match your DNS

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-resource-nginx
spec:
  rules:
  - host: nginx.dev.kube-g1.kubehol.net
    http:
      paths:
      - backend:
          serviceName: first-nginx-pod
          servicePort: 80
```

```
kubectl apply -f ingress-resource.yml
```

Let's take a look to make sure the Ingress rules were created successfully
```
kubectl describe ing ingress-resource-nginx
Name:             ingress-resource-nginx
Namespace:        default
Address:
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host                                 Path  Backends
  ----                                 ----  --------
  nginx.dev.kube-g1.kubehol.net        nginx-deploy-main:80 (10.233.127.16:80)
Annotations:
Events:  <none>
```

Now if everything is working correctly. Open your browser and you should see:

### Welcome to nginx!

<br>

Now for a final check, delete the `ingress-resource-nginx`.

```
kubectl delete ing ingress-resource-nginx
```

Refresh the webpage, it should stop working.

Recreate the rule again, and it should start routing traffic again.

```
kubectl apply -f ingress-resource.yml
```
