# Services and Networking

## 1 : Create a pod with image nginx called nginx and expose its port 80

```bash
k run nginx --image nginx --port 80 --expose=true

# or
k run nginx --image nginx --port 80
k expose pod nginx --port 80
```

## 2 : Confirm that ClusterIP has been created. Also check endpoints

```bash
k get svc nginx $o | grep -i clusterip

k get ep
```

## 3 : Get service's ClusterIP, create a temp busybox pod and 'hit' that IP with wget

```bash
k get svc nginx $o | grep -i clusterip

k run busybox --image=busybox --restart=Never -it --rm -- wget -qO- http://SVC_IP
```

## 4 : Convert the ClusterIP to NodePort for the same service and find the NodePort port. Hit service using Node's IP. Delete the service and the pod at the end.

```bash
k get svc nginx $o > files/06_04_svc.yaml

# Replace
#   type: ClusterIP --> NodePort
# Add
#   ports:
#   - ...
#     nodePort: 30100
k apply -f files/06_04_svc.yaml

k get nodes -o wide

# If you use Kind, execute it in the container : 
# docker exec -it CTN_ID sh
curl http://NODE_IP:30100 
```

## 5 : Create a deployment called foo using image 'dgkanatsios/simpleapp' (a simple server that returns hostname) and 3 replicas. Label it as 'app=foo'. Declare that containers in this pod will accept traffic on port 8080 (do NOT create a service yet)

```bash
k create deploy foo --image=dgkanatsios/simpleapp --replicas=3 --port=8080 $dry $o > files/06_05_deploy.yaml
# add the following
#   labels:
#     app: foo
k apply -f files/06_05_deploy.yaml

```

## 6 : Get the pod IPs. Create a temp busybox pod and try hitting them on port 8080

```bash
k get po -o wide -L app=foo
k run busybox --image=busybox --restart=Never -it --rm -- wget -qO- http://POD_IP:8080
```

## 7 : Create a service that exposes the deployment on port 6262. Verify its existence, check the endpoints

```bash
k expose deploy foo --port=6262 --target-port=8080
k get svc foo
k get ep foo
```

## 8 : Create a temp busybox pod and connect via wget to foo service. Verify that each time there's a different hostname returned. Delete deployment and services to cleanup the cluster

```bash
k run busybox --image=busybox --restart=Never -it --rm -- wget -qO- http://CLUSTER_IP:6262

# or
k run busybox --image=busybox --restart=Never -it --rm -- wget -qO- http://foo:6262

# or
k run busybox --image=busybox --restart=Never -it --rm -- wget -qO- http://foo.default:6262
```

## 9 : Create an nginx deployment of 2 replicas, expose it via a ClusterIP service on port 80. Create a NetworkPolicy so that only pods with labels 'access: granted' can access the deployment and apply it

```bash
k create deploy nginx --image=nginx --replicas=2
k expose deploy nginx --port=80 --type=ClusterIP

touch files/06_09_netpol.yaml
# add the following
# kind: NetworkPolicy
# apiVersion: networking.k8s.io/v1
# metadata:
#   name: access-nginx
# spec:
#   podSelector:
#     matchLabels:
#       app: nginx
#   ingress:
#   - from:
#     - podSelector:
#         matchLabels:
#           access: granted
k apply -f files/06_09_netpol.yaml
k get netpol

k get svc
k run busybox --image=busybox -l access=granted --restart=Never -it --rm -- wget -qO- http://nginx --timeout 2
k run busybox --image=busybox --restart=Never -it --rm -- wget -qO- http://nginx --timeout 2

# Tip : Ensure a CNI, like Calico, is well configured to enable NetworkPolicy support
# https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart
# https://docs.tigera.io/calico/latest/network-policy/get-started/kubernetes-policy/kubernetes-policy-basic
```
