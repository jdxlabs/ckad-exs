

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
