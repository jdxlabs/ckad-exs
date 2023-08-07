

Create a namespace called 'ns1' and a pod with image nginx called nginx on this namespace

```
k create ns ns1

k run nginx --image=nginx --restart=Never -n ns1

k config set-context --current --namespace ns1
k get po
```