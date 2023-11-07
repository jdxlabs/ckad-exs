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
k create ns one
k create quota my-rq --namespace=one --hard=requests.cpu=1,requests.memory=1Gi,limits.cpu=2,limits.memory=2Gi

```

## 16 : Attempt to create a pod with resource requests cpu=2, memory=3Gi and limits cpu=3, memory=4Gi in namespace one

```bash
k run nginx --image=nginx -n one $dry $o > files/04_16_po.yaml

# add the following
#   resources:
#     requests:
#       cpu: 2
#       memory: 3Gi
#     limits:
#       cpu: 3
#       memory: 4Gi

# Expected error : 
# Error from server (Forbidden): error when creating "files/04_16_po.yaml": pods "nginx" is forbidden: exceeded quota: my-rq, requested: limits.cpu=3,limits.memory=4Gi,requests.cpu=2,requests.memory=3Gi, used: limits.cpu=0,limits.memory=0,requests.cpu=0,requests.memory=0, limited: limits.cpu=2,limits.memory=2Gi,requests.cpu=1,requests.memory=1Gi
```

## 17 : Create a pod with resource requests cpu=0.5, memory=1Gi and limits cpu=1, memory=2Gi in namespace one

```bash
k run nginx --image=nginx -n one $dry $o > files/04_17_po.yaml

# add the following
#   resources:
#     requests:
#       cpu: 0.5
#       memory: 1Gi
#     limits:
#       cpu: 1
#       memory: 2Gi

# k get quota
# NAME    AGE   REQUEST                                          LIMIT
# my-rq   37s   requests.cpu: 500m/1, requests.memory: 1Gi/1Gi   limits.cpu: 1/2, limits.memory: 2Gi/2Gi
```


# Secrets

## 18 : Create a secret called mysecret with the values password=mypass

```bash
k create secret generic mysecret --from-literal=password=mypass
```

## 19 : Create a secret called mysecret2 that gets key/value from a file

```bash
touch files/04_19_secret.txt
echo -n admin > files/04_19_secret-username

k create secret generic mysecret2 --from-file=files/04_19_secret-username

```

## 20 : Get the value of mysecret2

```bash
k get secret mysecret2 $o
echo -n YWRtaW4= | base64 -d

# or
k get secret mysecret2 -o jsonpath='{.data.04_19_secret-username}' | base64 -d

# or, with template
k get secret mysecret2 --template '{{.data}}'

# or, with jq
k get secret mysecret2 -o json | jq -r .data
```

## 21 : Create an nginx pod that mounts the secret mysecret2 in a volume on path /etc/foo

```bash
k run nginx --image=nginx $dry $o > files/04_21_po.yaml

# add the following
#   volumes:
#   - name: myvolume
#     secret:
#       secretName: mysecret2
#   containers:
#   - ...
#     volumeMounts:
#     - name: myvolume
#       mountPath: /etc/foo
```

## 22 : Delete the pod you just created and mount the variable 'username' from secret mysecret2 onto a new nginx pod in env variable called 'USERNAME'

```bash
k delete po nginx $f

k run nginx --image=nginx $dry $o > files/04_22_po.yaml
# add the following
#   containers:
#   - ...
#     env:
#     - name: USERNAME
#       valueFrom:
#         secretKeyRef:
#           name: mysecret2
#           key: 04_19_secret-username
```

## 23 : Create a Secret named 'ext-service-secret' in the namespace 'secret-ops'. Then, provide the key-value pair API_KEY=LmLHbYhsgWZwNifiqaRorH8T as literal

```bash
k create ns secret-ops
kn secret-ops
k create secret generic ext-service-secret --from-literal=API_KEY=LmLHbYhsgWZwNifiqaRorH8T
k get secrets ext-service-secret $o -o jsonpath="{.data.API_KEY}"
```

## 24 : Consuming the Secret. Create a Pod named 'consumer' with the image 'nginx' in the namespace 'secret-ops' and consume the Secret as an environment variable. Then, open an interactive shell to the Pod, and print all environment variables

```bash
kn secret-ops
k run consumer --image=nginx $dry $o -n secret-ops > files/04_24_po.yaml
# add the following
#   containers:
#   - ...
#     env:
#     - name: API_KEY
#       valueFrom:
#         secretKeyRef:
#           name: ext-service-secret
#           key: API_KEY

k exec -it consumer -- sh -c "env | grep API_KEY"
```

## 25 : Create a Secret named 'my-secret' of type 'kubernetes.io/ssh-auth' in the namespace 'secret-ops'. Define a single key named 'ssh-privatekey', and point it to the file 'id_rsa' in this directory

```bash
kn secret-ops
ssh-keygen -t rsa -b 2048 -n '' -f files/04_25_id_rsa
k create secret generic my-secret --type=kubernetes.io/ssh-auth -n secret-ops --from-file=ssh-privatekey=files/04_25_id_rsa
```

## 26 : Create a Pod named 'consumer' with the image 'nginx' in the namespace 'secret-ops', and consume the Secret as Volume. Mount the Secret as Volume to the path /var/app with read-only access. Open an interactive shell to the Pod, and render the contents of the file

```bash
kn secret-ops
k run consumer --image=nginx $dry $o -n secret-ops > files/04_26_po.yaml

# add the following
#   volumes:
#   - name: ssh
#     secret:
#       secretName: my-secret
#   containers:
#   - ...
#     volumeMounts:
#     - name: ssh
#       mountPath: /var/app
#       readOnly: true

k exec -it consumer -- sh -c "cat /var/app/ssh-privatekey"

```


# ServiceAccounts

## 27 : See all the service accounts of the cluster in all namespaces

```bash
k get sa
```

## 28 : Create a new serviceaccount called 'myuser'

```bash
k create sa myuser
```

## 29 : Create an nginx pod that uses 'myuser' as a service account

```bash
k run nginx --image=nginx $dry $o > files/04_29_po.yaml
# add the following
#  spec:
#    serviceAccountName: myuser
```

## 30 : Generate an API token for the service account 'myuser'

```bash
k create token myuser
```
