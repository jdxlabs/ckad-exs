# Services and Networking

## 1 : Create a pod with image nginx called nginx and expose its port 80

```bash
k run nginx --image nginx --port 80 --expose=true
```

## 2 : Confirm that ClusterIP has been created. Also check endpoints

```bash
k get svc nginx $o | grep -i clusterip

k get ep
```

## 3 : Get service's ClusterIP, create a temp busybox pod and 'hit' that IP with wget

```bash
k get svc nginx $o | grep -i clusterip

k run busybox --image=busybox --restart=Never -it --rm -- wget -O- http://10.96.128.56
```

## 4 : Convert the ClusterIP to NodePort for the same service and find the NodePort port. Hit service using Node's IP. Delete the service and the pod at the end.

```bash
```

## 5 : Create a deployment called foo using image 'dgkanatsios/simpleapp' (a simple server that returns hostname) and 3 replicas. Label it as 'app=foo'. Declare that containers in this pod will accept traffic on port 8080 (do NOT create a service yet)

```bash
```

## 6 : Get the pod IPs. Create a temp busybox pod and try hitting them on port 8080

```bash
```

## 7 : Create a service that exposes the deployment on port 6262. Verify its existence, check the endpoints

```bash
```

## 8 : Create a temp busybox pod and connect via wget to foo service. Verify that each time there's a different hostname returned. Delete deployment and services to cleanup the cluster

```bash
```

## 9 : Create an nginx deployment of 2 replicas, expose it via a ClusterIP service on port 80. Create a NetworkPolicy so that only pods with labels 'access: granted' can access the deployment and apply it

```bash
```
