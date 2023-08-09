

## 1 : Create a namespace called 'ns1' and a pod with image nginx called nginx on this namespace

```bash
k create ns ns1
k config set-context --current --namespace ns1

k run nginx --image=nginx
k get po
```

## 2 : Create the pod that was just described using YAML
```bash
export dry=--dry-run=client
export o=-oyaml

k run nginx --image=nginx $dry $o > files/01_02_po.yml

```

## 3 : Create a busybox pod (using kubectl command) that runs the command "env". Run it and see the output
```bash
k run busybox --image=busybox --restart=Never --command -it --rm -- env
```

## 4 : Create a busybox pod (using YAML) that runs the command "env". Run it and see the output
```bash
k run busybox --image=busybox --restart=Never $dry $o -- env > files/01_04_po.yml
```

## 5 : Get the YAML for a new namespace called 'ns2' without creating it
```bash
k create ns ns2 $dry $o
```

## 6 : Get the YAML for a new ResourceQuota called 'rq1' with hard limits of 1 CPU, 1G memory and 2 pods without creating it
```bash
k create quota rq1 --hard=cpu=1,memory=1G,pods=2 $dry $o > 01_06_rq.yml
```

## 7 : Get pods on all namespaces
```bash
k get po -A
```

## 8 : Create a pod with image nginx called nginx and expose traffic on port 80
```bash
k run nginx --image=nginx $dry $o --port=80 > files/01_08_po.yml
```

## 9 : Change pod's image to nginx:1.7.1. Observe that the container will be restarted as soon as the image gets pulled
```bash
k set image pod/nginx nginx=nginx:1.7.1

# the pod's image is automatically updated
k describe po nginx
k get po
```

## 10 : Get nginx pod's ip created in previous step, use a temp busybox image to wget its '/'
```bash
k get po -o wide
k run tmp --image=nginx:alpine --rm -it --restart=Never -- curl http://<current-ip>
```

## 11 : Get pod's YAML
```bash
k get po nginx $o
```

## 12 : Get information about the pod, including details about potential issues (e.g. pod hasn't started)
```bash
k describe po nginx
```

## 13 : Get pod logs
```bash
k logs nginx
```

## 14 : If pod crashed and restarted, get logs about the previous instance
```bash
k logs nginx -p
```

## 15 : Execute a simple shell on the nginx pod
```bash
k exec -it nginx -- sh
```

## 16 : Create a busybox pod that echoes 'hello world' and then exits
```bash
k run tmp --image=busybox -it --restart=Never -- sh -c 'echo "hello world"'
```

## 17 : Do the same, but have the pod deleted automatically when it's completed
```bash
k run tmp --image=busybox --rm -it --restart=Never -- sh -c 'echo "hello world"'
```

## 18 : Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence within the pod
```bash
k run nginx --image=nginx --rm -it --env="var1=val1" --restart=Never -- sh -c 'echo $var1'
```
