# Liveness, readiness and startup probes

## 1 : Create an nginx pod with a liveness probe that just runs the command 'ls'. Save its YAML in pod.yaml. Run it, check its probe status, delete it.

```bash
k run nginx --image=nginx --restart=Never $dry $o > files/05_01_pod.yaml
# add the following
#  containers:
#  - ...
#    livenessProbe:
#      exec:
#        command:
#        - ls

k describe po nginx | grep -i liveness
#   Liveness:       exec [ls] delay=0s timeout=1s period=10s #success=1 #failure=3

k delete po nginx $f
```

## 2 : Modify the pod.yaml file so that liveness probe starts kicking in after 5 seconds whereas the interval between probes would be 5 seconds. Run it, check the probe, delete it.

```bash
k run nginx --image=nginx --restart=Never $dry $o > files/05_02_pod.yaml
# add the following
#  containers:
#  - ...
#    livenessProbe:
#      exec:
#        command:
#        - ls
#      initialDelaySeconds: 5
#      periodSeconds: 5

k describe po nginx | grep -i liveness                                  
#   Liveness:       exec [ls] delay=5s timeout=1s period=5s #success=1 #failure=3

k delete po nginx $f
```

## 3 : Create an nginx pod (that includes port 80) with an HTTP readinessProbe on path '/' on port 80. Again, run it, check the readinessProbe, delete it.

```bash
k run nginx --image=nginx --restart=Never $dry $o > files/05_03_pod.yaml
# add the following
#  containers:
#  - ...
#    readinessProbe:
#      httpGet:
#        path: /
#        port: 80
#        scheme: HTTP

k describe po nginx | grep -i -A10 readiness
```

## 4 : Lots of pods are running in qa,alan,test,production namespaces. All of these pods are configured with liveness probe. Please list all pods whose liveness probe are failed in the format of <namespace>/<pod name> per line.

```bash
k get events -o json | jq -r '.items[] | select(.message | contains("failed liveness probe")).involvedObject | .namespace + "/" + .name'
```


# Logging

## 5 : Create a busybox pod that runs i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done. Check its logs

```bash
k run busybox --image=busybox $dry $o -- sh -c 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done' > files/05_05_busybox.yaml

k logs -f busybox
```


# Debugging

## 6 : Create a busybox pod that runs 'ls /notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod

```bash
k run busybox --image=busybox --restart=Never $dry $o -- sh -c 'ls /notexist' > files/05_06_busybox.yaml

k logs busybox
# ls: /notexist: No such file or directory
```

## 7 : Create a busybox pod that runs 'notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod forcefully with a 0 grace period

```bash
k run busybox --image=busybox --restart=Never $dry $o -- sh -c 'notexist' > files/05_07_busybox.yaml

k logs busybox
# sh: notexist: not found

k delete po busybox $f
```

## 8 : Get CPU/memory utilization for nodes (metrics-server must be running)

```bash
# setup metrics-server
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml >> files/05_08_metrics_server.yaml
# add insecure TLS parameter to the yaml file
#   metadata:
#       labels:
#         k8s-app: metrics-server
#     spec:
#       containers:
#       - args:
#         - --cert-dir=/tmp
#         - --secure-port=4443
#         - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
#         - --kubelet-use-node-status-port
#         - --kubelet-insecure-tls   <==== Add this
#         - --metric-resolution=15s
#         image: k8s.gcr.io/metrics-server/metrics-server:v0.6.2
#         imagePullPolicy: IfNotPresent
#         livenessProbe:
#           failureThreshold: 3
#           httpGet:
#             path: /livez
k apply -f files/05_08_metrics_server.yaml

k top node
```
