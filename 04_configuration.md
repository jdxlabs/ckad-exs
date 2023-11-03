# ConfigMaps

## 1 : Create a configmap named config with values foo=lala,foo2=lolo

```bash
k create configmap config --from-literal=foo=lala --from-literal=foo2=lolo
```

## 2 : Display its values

```bash
k describe cm config

# or
k get cm config $o
```

## 3 : Create and display a configmap from a file

```bash
echo -e "foo3=lili\nfoo4=lele" > files/04_03_configmap.txt
k create configmap config-txt --from-file=files/04_03_configmap.txt
```

## 4 : Create and display a configmap from a .env file

```bash
echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > files/04_04_configmap.env
k create configmap config-env --from-env-file=files/04_04_configmap.env
```

## 5 : Create and display a configmap from a file, giving the key 'special'

```bash
echo -e "var3=val3\nvar4=val4" > files/04_05_configmap.txt
k create configmap config2-txt --from-file=special=files/04_05_configmap.txt
```

## 6 : Create a configMap called 'options' with the value var5=val5. Create a new nginx pod that loads the value from variable 'var5' in an env variable called 'option'

```bash
k create configmap options --from-literal=var5=val5

k run nginx --image=nginx --restart=Never --overrides='{"spec":{"containers":[{"name":"nginx","env":[{"name":"option","valueFrom":{"configMapKeyRef":{"name":"options","key":"var5"}}}]}]}}'

# or 

k run nginx --image=nginx --restart=Never $dry $o > files/04_06_po.yaml
# add the following
#   env:
#   - name: option
#     valueFrom:
#         configMapKeyRef:
#             name: options
#             key: var5
```

## 7 : Create a configMap 'anotherone' with values 'var6=val6', 'var7=val7'. Load this configMap as env variables into a new nginx pod

```bash
k create configmap anotherone --from-literal=var6=val6 --from-literal=var7=val7

k run nginx --image=nginx --restart=Never $dry $o > files/04_07_po.yaml
# add the following
#   envFrom:
#   - configMapRef:
#       name: anotherone
```


## 8 : Create a configMap 'cmvolume' with values 'var8=val8', 'var9=val9'. Load this as a volume inside an nginx pod on path '/etc/lala'. Create the pod and 'ls' into the '/etc/lala' directory.

```bash
k create configmap cmvolume --from-literal=var8=val8 --from-literal=var9=val9

k run nginx --image=nginx --restart=Never $dry $o > files/04_08_po.yaml
# add the following
#   volumes:
#   - name: myvolume
#     configMap:
#       name: cmvolume
#   containers:
#   - ...
#     volumeMounts:
#     - name: myvolume
#       mountPath: /etc/lala
```


# SecurityContext

## 9 : Create the YAML for an nginx pod that runs with the user ID 101. No need to create the pod

```bash
k run nginx --image=nginx --restart=Never $dry $o > files/04_09_po.yaml
# add the following
#   securityContext:
#     runAsUser: 101
```

## 10 : Create the YAML for an nginx pod that has the capabilities "NET_ADMIN", "SYS_TIME" added to its single container

```bash
k run nginx --image=nginx --restart=Never $dry $o > files/04_10_po.yaml
# add the following (on the single container)
#   securityContext:
#     capabilities:
#       add:
#       - NET_ADMIN
#       - SYS_TIME
```


# Resource requests and limits

## 11 : Create an nginx pod with requests cpu=100m,memory=256Mi and limits cpu=200m,memory=512Mi

```bash
k run nginx --image=nginx --restart=Never $dry $o > files/04_11_po.yaml
# add the following
#   resources:
#     requests:
#       cpu: 100m
#       memory: 256Mi
#     limits:
#       cpu: 200m
#       memory: 512Mi
```


# Limit Ranges

## 12 : Create a namespace with limit range

```bash
k create ns one
kn one

touch files/04_12_limitrange.yaml
# get the example in documentation (max / min memory)

k apply -f files/04_12_limitrange.yaml
```

## 13 : Describe the namespace limitrange

```bash
k get limitrange
k describe limitrange memory-resource-constraint
```

## 14 : Create a pod with resources requests memory = half of max memory constraint in namespace

```bash
k run nginx --image=nginx --restart=Never $dry $o > files/04_14_po.yaml

# add the following
#   resources:
#     requests:
#       memory: 250Mi

k apply -f files/04_14_po.yaml


k describe po nginx | grep -i request -n10
# or (more precise)
k describe po nginx | grep -i request -A1
```


# Resource Quotas

## 15 : Create ResourceQuota in namespace one with hard requests cpu=1, memory=1Gi and hard limits cpu=2, memory=2Gi

```bash

```

## 16 : Attempt to create a pod with resource requests cpu=2, memory=3Gi and limits cpu=3, memory=4Gi in namespace one

```bash
```

## 17 : Create a pod with resource requests cpu=0.5, memory=1Gi and limits cpu=1, memory=2Gi in namespace one

```bash
```


# Secrets

## 18 : Create a secret called mysecret with the values password=mypass

```bash
```

## 19 : Create a secret called mysecret2 that gets key/value from a file

```bash
```

## 20 : Get the value of mysecret2

```bash
```

## 21 : Create an nginx pod that mounts the secret mysecret2 in a volume on path /etc/foo

```bash
```

## 22 : Delete the pod you just created and mount the variable 'username' from secret mysecret2 onto a new nginx pod in env variable called 'USERNAME'

```bash
```

## 23 : Create a Secret named 'ext-service-secret' in the namespace 'secret-ops'. Then, provide the key-value pair API_KEY=LmLHbYhsgWZwNifiqaRorH8T as literal

```bash
```

## 24 : Consuming the Secret. Create a Pod named 'consumer' with the image 'nginx' in the namespace 'secret-ops' and consume the Secret as an environment variable. Then, open an interactive shell to the Pod, and print all environment variables

```bash
```

## 25 : Create a Secret named 'my-secret' of type 'kubernetes.io/ssh-auth' in the namespace 'secret-ops'. Define a single key named 'ssh-privatekey', and point it to the file 'id_rsa' in this directory

```bash
```

## 26 : Create a Pod named 'consumer' with the image 'nginx' in the namespace 'secret-ops', and consume the Secret as Volume. Mount the Secret as Volume to the path /var/app with read-only access. Open an interactive shell to the Pod, and render the contents of the file

```bash
```


# ServiceAccounts

## 27 : See all the service accounts of the cluster in all namespaces

```bash
```

## 28 : Create a new serviceaccount called 'myuser'

```bash
```

## 29 : Create an nginx pod that uses 'myuser' as a service account

```bash
```

## 30 : Generate an API token for the service account 'myuser'

```bash
```
