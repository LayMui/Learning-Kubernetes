# From Kubernetes 101 & Workshop

These demo docs at https://is.gd/sinivi.

Session slides at https://is.gd/uquzut.

Video at https://youtu.be/dDLrvc9IVq4.

## Setup

Run Kubernetes locally with [Docker Desktop](https://www.docker.com/products/docker-desktop/) or [kind](https://kind.sigs.k8s.io/docs/user/quick-start/), or use a playground like [Play with Kubernetes](https://labs.play-with-k8s.com) or [Katacoda](https://www.katacoda.com/courses/kubernetes/playground).

```
cd 202005-kubernetes-101/
```

## Demo 1 - Sleep

> Walk through [sleep.yaml](sleep.yaml).
All k8s yml start with the api version for its schema
kind is the kind of object you defined which is deployment
that lives in this apps/v1 namespace
K8s has some many objects, so it's easier to break into namespace

the metadata name tell me the name of the object which is sleep
the metadata label is the key-value pair applied to different things to help you manage your object.
metadata is the information about the deployment
spec is the specification of the deployment itself
the deployment object is the one that ultimately created my pods for me
the pod specification is at
spec: 
    containers:
        -name: sleep
        image: kiamol/chp03-sleep
the deployment will be labelled demo: kube101
and pod will be labelled app: sleep
K8s uses label selector to find resources
deployment owns the pod with label app:sleep
the yml is your desired state and run use kubectl to create the deployment object defined in the yml file.
I could have use docker run to run the image but now I am asking Kubernetes to run and manage the container running with the pod for me.
the kubectl describe command give me more information such as setup of the pod and internal detail like the IP address of the pod and the events in it.
kubetcl exec is to connect to the pod just like running docker exec command. It will run the command hostname inside the pod.

Simple Pod:

```
kubectl apply -f sleep.yaml

kubectl get pods --show-labels

kubectl describe pod -l app=sleep

kubectl exec deploy/sleep -- hostname
```

Pod restart:

```
kubectl exec deploy/sleep -- killall5

kubectl get pods
```

Pod replacement:

```
kubectl delete pods -l app=sleep

kubectl get pods
```

## Demo 2 - Web
> Walk through [bb-web-service.yaml](web/bb-web-service.yaml) and [bb-web.yaml](web/bb-web.yaml).
The metadata name: bb-web is also the DNS name
The Services is just a network layer and it does not have a container
When there is traffic coming to my service, it's going to send that traffic onto that pod with the selector labelled app:bb-web
so the selector decouple the network layer from the compute layer
This service is a loadbalancer 
and I will get a public IP address when I deploy this service and that IP address is where I configure my DNS name
The cluster IP address of the DNS service loadbalancer is fixed
it won't change unless I delete the service and create a new one.
This is the internal IP address of the cluster


Deploy all:

```
kubectl apply -f web/

kubectl get svc
```

Check status:

```
kubectl get pods -l app=bb-web

kubectl logs -l app=bb-web
```

Check network:

```
kubectl get svc bb-web

kubectl exec deploy/sleep -- nslookup bb-web

kubectl exec deploy/sleep -- curl http://bb-web:8080
```

## Demo 3 - Database

> Walk through [bb-db-service.yaml](db/bb-db-service.yaml), [bb-db.yaml](db/bb-db.yaml) and [bb-db-secret.yaml](db/bb-db-secret.yaml).
bb-db-service-.yaml: is a ClusterIP type of service. I can only access from my pod within my cluster.
I will find my pod with the selector label app with the value bb-db
and the service will find them and send traffic in.
It used DNS name bb-db as it's database server and it's used by the web app only

Deploy all:

```
kubectl apply -f db/

kubectl get svc

kubectl get pods -l app=bb-db

kubectl logs -l app=bb-db
```

Check app status:

```
kubectl get pods -l app=bb-web
```

Get URL & browse:

```
kubectl get svc bb-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8080'
```

kubectl get svc bb-web -o jsonpath='{}'
It will get back the actual data of my object in JSON
because we are using JSON path, we can craft the bit that I am interested in
This mean get me the address of the loadBalancer and wrap it in the HTTP string
it will get http://localhost:8080
Different loadBalancer has different way of showing the IP address
Scale up:

```
kubectl scale deploy/bb-web --replicas=3

kubectl get pods -l app=bb-web

kubectl get endpoints bb-web
```

> Browse and refresh lots

Check load balancing:

```
kubectl logs -l app=bb-web --tail 1
```

## Teardown

All the objects are labelled with `demo=kube101`, so we can delete them all using a label selector.

```
kubectl get all -l demo=kube101

kubectl delete all -l demo=kube101
```