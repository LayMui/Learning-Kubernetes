### Microservices

These simple microservices enable us to **focus on** learning the tools - Docker, Kubernetes, CI, CD and  build the infrastructure needed around typical microservices.

### Currency Exchange Service

If you ask it the value of 1 USD in INR, or 1 Australian Dollar in INR, the Currency Exchange Service answers 
- 1 USD is 60 INR
- 1 Australian Dollars is 50 INR. 

http://localhost:8000/currency-exchange/from/EUR/to/INR

```json
{
  "id": 10002,
  "from": "EUR",
  "to": "INR",
  "conversionMultiple": 75.00,
  "exchangeEnvironmentInfo": "37f1ad927c6e v1 27c6e"
}
```

### Currency Conversion

Currency Conversion Service is used to convert a bucket of currencies. If you want to find the value of 10 USD, Currency Conversion Service returns 600. 
- **STEP 1** : Currency Conversion Service calls the Currency Exchange Service for the value of 1 USD. It gets a response back saying 60.
- **STEP 2** : The Currency Conversion Service then multiplies 10 by 60, and returns 600 back. 

http://localhost:8100/currency-conversion/from/EUR/to/INR/quantity/10

```json
{
  "id": 10002,
  "from": "EUR",
  "to": "INR",
  "conversionMultiple": 75.00,
  "quantity": 10,
  "totalCalculatedAmount": 750.00,
  "exchangeEnvironmentInfo": "37f1ad927c6e v1 27c6e",
  "conversionEnvironmentInfo": "fb6316b5713d v1 5713d"
}
```

#### How does Currency Conversion know the location of Currency Exchange?
- You don't want to HARDCODE
- Configure an Environment Variable - `CURRENCY_EXCHANGE_SERVICE_HOST`
- --env CURRENCY_EXCHANGE_SERVICE_HOST=http://currency-exchange

Instructions to run:
cd microservices/02-currency-conversion-microservices-basic
kubectl apply -f deployment.yaml
cd microservices/01-currency-exchange-microservice-basic/kubernetes
kubectl apply -f deployment.yaml

kubectl get services
NAME                  TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)          AGE
currency-conversion   LoadBalancer   10.16.9.158   35.223.57.65    8100:30627/TCP   4m30s
currency-exchange     LoadBalancer   10.16.8.191   34.67.207.199   8000:30049/TCP   2m13s
kubernetes            ClusterIP      10.16.0.1     <none>          443/TCP          41h

kubectl get svc -watch
Error: unknown shorthand flag: 'a' in -atch
See 'kubectl get --help' for usage.
Lays-MBP:kubernetes laymui$ kubectl get svc --watch
NAME                  TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)          AGE
currency-conversion   LoadBalancer   10.16.9.158   35.223.57.65    8100:30627/TCP   5m11s
currency-exchange     LoadBalancer   10.16.8.191   34.67.207.199   8000:30049/TCP   2m54s
kubernetes            ClusterIP      10.16.0.1     <none>          443/TCP          41h


To see the currency exchange URL: 
http://localhost:8000/currency-exchange/from/EUR/to/INR
replace localhost with 34.67.207.199  
http://34.67.207.199:8000/currency-exchange/from/EUR/to/INR


To see the currency conversion URL:
http://localhost:8100/currency-conversion/from/EUR/to/INR/quantity/10
replace locahost with 35.223.57.65 (Note: didn't work!)
There was an unexpected error (type=Internal Server Error, status=500).
Connection refused (Connection refused) executing GET http://localhost:8000/currency-exchange/from/EUR/to/INR

kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
currency-conversion-5696c595f4-lx94m   1/1     Running   0          75s
currency-conversion-5696c595f4-xqblk   1/1     Running   0          28s
currency-conversion-6c47fdb64f-j4w7v   1/1     Running   0          17m
currency-exchange-77c7759dc4-d9bhp     1/1     Running   0          15m

When a service is being launched up, the information about all the other services which are currently live, was passed on to this specific service as environment variables by kubernetes.

Let's look at specific CURRENCY_EXCHANGE_SERVICE_HOST.
how is this name form?

kubectl get services
NAME                  TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)          AGE
currency-conversion   LoadBalancer   10.16.9.158   35.223.57.65    8100:30627/TCP   33m
currency-exchange     LoadBalancer   10.16.8.191   34.67.207.199   8000:30049/TCP   30m
kubernetes            ClusterIP      10.16.0.1     <none>          443/TCP          41h

The env variable is created based on the service name
name of the service is capitalized and hyphen replaced by underscore
currency-exchange -> CURRENCY_EXCHANGE_SERVICE_HOST
That is why the 2 microservices can talk to each other at start up

Configure the env variable to the url value
 env:
          - name: CURRENCY_EXCHANGE_SERVICE_HOST
            value: http://currency-exchange

more reliable approach becos this url is always available
this work becos of the automatic service discovery by K8s
I want to find all the instances of this service, This is called Service Discovery
K8s comes up with an automated service discovery system called DNS
if you use this kind of a URI, you also get load balancing for free
To find out the number of instances available
kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
currency-conversion-5696c595f4-lx94m   1/1     Running   0          39m
currency-conversion-5696c595f4-xqblk   1/1     Running   0          38m
currency-exchange-77c7759dc4-d9bhp     1/1     Running   0          52m

We want to reduce the number of instance in currency conversion to 1 and increase currency exchange to 2 and see if the load balancing occur automatically

http://34.67.207.199:8000/currency-exchange/from/EUR/to/INR
http://35.223.57.65:8100/currency-conversion/from/EUR/to/INR/quantity/10

Configuration Management:
A big headache to store different configuration for different microservices
That's why we need a centralised configration 
Kubernetes provides centralised config using ConfigMap using yaml file 00-configmap-currency-conversion.yaml

kubectl apply -f 00-configmap-currency-conversion.yaml
kubectl get configMaps
kubectl describe configmap currency-conversion-config-map

Now we want our currency conversion service to use the value from the
config map (from CURRENCY_EXCHANGE_SERVICE_HOST)
Go to the deployment.yaml to specify to pick up from the configMap
 valueFrom: 
      configMapKeyRef:
        key: CURRENCY_EXCHANGE_SERVICE_HOST
        name: currency-conversion-config-map


kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
currency-conversion-5696c595f4-lx94m   1/1     Running   0          53m
currency-conversion-5b7d44964b-jrdlq   1/1     Running   0          25s
currency-exchange-77c7759dc4-49kb6     1/1     Running   0          11m
currency-exchange-77c7759dc4-d9bhp     1/1     Running   0          66m

kubectl logs -f currency-exchange-77c7759dc4-49kb6

Now change the 00-configmap-currency-conversion.yaml
and run kubectl appl -f 00-configmap-currency-conversion.yaml
and run kubectl describe configMap
ame:         currency-conversion-config-map
Namespace:    default
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","data":{"CURRENCY_EXCHANGE_SERVICE_HOST":"http://currency-exchange1"},"kind":"ConfigMap","metadata":{"annotations":{},"...

Data
====
CURRENCY_EXCHANGE_SERVICE_HOST:
----
http://currency-exchange1
Events:  <none>

kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
currency-conversion-5b7d44964b-jrdlq   1/1     Running   0          6m11s
currency-exchange-77c7759dc4-49kb6     1/1     Running   0          17m
currency-exchange-77c7759dc4-d9bhp     1/1     Running   0          72m
kubectl delete pod currency-conversion-5b7d44964b-jrdlq
pod "currency-conversion-5b7d44964b-jrdlq" deleted

This will launch up a new pod

kubectl get podsNAME                                   READY   STATUS    RESTARTS   AGE
currency-conversion-5b7d44964b-tfgqk   1/1     Running   0          31s
currency-exchange-77c7759dc4-49kb6     1/1     Running   0          18m
currency-exchange-77c7759dc4-d9bhp     1/1     Running   0          73m

http://35.223.57.65:8100/currency-conversion/from/EUR/to/INR/quantity/10
This application has no explicit mapping for /error, so you are seeing this as a fallback.

Wed Jul 22 06:50:24 GMT 2020
There was an unexpected error (type=Internal Server Error, status=500).
currency-exchange1 executing GET http://currency-exchange1:8000/currency-exchange/from/EUR/to/INR

because the http://currency-exchange1 is not the right value

revert back
and do a kubectl get pods and delete the currency-exchange pod


Convert Load balancer to NodePort
kubectl get svc
NAME                  TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)          AGE
currency-conversion   LoadBalancer   10.16.9.158   35.223.57.65    8100:30627/TCP   84m
currency-exchange     LoadBalancer   10.16.8.191   34.67.207.199   8000:30049/TCP   81m
kubernetes            ClusterIP      10.16.0.1     <none>          443/TCP          42h

Don't want to create multiple load balancer for multiple microservices becos Load Balancer is expensive due to their HA
So we use Ingress
We switch the Loadbalancer to NodePort and create ingress to route
the request to the appropriate services based on the URL
Go to the deployment.yaml file and replace LoadBalancer with NodePort
kubectl apply -f deployment.yaml
kubectl get svc
NAME                  TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
currency-conversion   NodePort    10.16.9.158   <none>        8100:30627/TCP   93m
currency-exchange     NodePort    10.16.8.191   <none>        8000:30049/TCP   91m
kubernetes            ClusterIP   10.16.0.1     <none>        443/TCP          42h

The Node is exposed on every port
and we can use that port to talk to that service
A node is Virtual Server where your pod is running

We want to create ingress on top of this to route the request 
to the appropriate microservices based on the url
kubectl apply -f ingress.yaml

Anything starting with */currency-exchange/ will get route to the currency-exchange service
similarly with currency-conversion
The name of the ingress is gateway-ingress 

To redirect the url as it is 
nginx.ingress.kubernetes.io/rewrite-target: /

Go to google cloud - Services -ingress
Ingress LoadBalancer IP 35.241.14.82
copy this IP address and replace http://34.67.207.199:8000/ (including the port)
http://35.241.14.82/currency-exchange/from/EUR/to/INR

similarly for currency conversion
http://35.241.14.82/currency-conversion/from/EUR/to/INR/quantity/10

We are now hitting the microservices through the ingress
the request goes to the ingress and ingress look at the url
and see the url pattern and that it start with current exchange and then redirect
the that specific service
If you need to create another microservice, just create a deployment and add
to the ingress.yaml so you do not need any load balancer

Delete the cluster (whenever you no longer need it)

