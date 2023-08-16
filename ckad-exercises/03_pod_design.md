# A. Labels And Annotations

## 1 : Create 3 pods with names nginx1,nginx2,nginx3. All of them should have the label app=v1

```bash
k run nginx1 --image=nginx --restart=Never --labels=app=v1
k run nginx2 --image=nginx --restart=Never --labels=app=v1
k run nginx3 --image=nginx --restart=Never --labels=app=v1
# k delete po --all
```

## 2 : Show all labels of the pods

```bash
k get po --show-labels
```

## 3 : Change the labels of pod 'nginx2' to be app=v2

```bash
k label po nginx2 app=v2 --overwrite
```

## 4 : Get the label 'app' for the pods (show a column with APP labels)

```bash
k get po -L app
```

## 5 : Get only the 'app=v2' pods

```bash
k get po -L app -l app=v2
```

## 6 : Add a new label tier=web to all pods having 'app=v2' or 'app=v1' labels

```bash
k label po -l "app in(v1,v2)" tier=web
k get po -L app,tier
```

## 7 : Add an annotation 'owner: marketing' to all pods having 'app=v2' label

```bash
k annotate po -l app=v2 owner=marketing
k annotate po --all --list
```

## 8 : Remove the 'app' label from the pods we created before

```bash
k label po --all app-
k get po -L app,tier
```

## 9 : Annotate pods nginx1, nginx2, nginx3 with "description='my description'" value

```bash
k annotate po nginx1 nginx2 nginx3 description='my description'
```

## 10 : Check the annotations for pod nginx1

```bash
k describe po nginx1 | grep -i -A 10 annotat
```

## 11 : Remove the annotations for these three pods

```bash
k annotate po nginx1 nginx2 nginx3 description-
```

## 12 : Remove these pods to have a clean state in your cluster

```bash
k delete po --all $f
```

# B. Pod Placement

## 1 : Create a pod that will be deployed to a Node that has the label 'accelerator=nvidia-tesla-p100'

```bash
# apply a label to a node
k label node kind1-control-plane accelerator=nvidia-tesla-p100

# show labels of nodes
k get nodes --show-labels

k run nginx --image=nginx --restart=Never --labels=accelerator=nvidia-tesla-p100
# it runs even if the node isn't labeled
```

## 2 : Taint a node with key tier and value frontend with the effect NoSchedule. Then, create a pod that tolerates this taint.

```bash
k taint node kind1-control-plane tier=frontend:NoSchedule

# show taints of nodes
k get nodes -o json | jq '.items[].spec.taints'

k run nginx --image=nginx $dry $o > files/03_b_02_po.yml

# add toleration to nginx.yaml, after the containers section
tolerations:
- key: "tier"
  operator: "Equal"
  value: "frontend"
  effect: "NoSchedule"

k apply -f files/03_b_02_po.yml
```

## 3 : Create a pod that will be placed on node controlplane. Use nodeSelector and tolerations.

```bash

```

# Deployments

## 1 : 

```bash

```

## 2 : 

```bash
```

## 3 : 

```bash
```

## 4 : 

```bash
```

## 5 : 

```bash
```

## 6 : 

```bash
```

## 7 : 

```bash
```

## 8 : 

```bash
```

## 9 : 

```bash
```

# Jobs

## 1 : 

```bash

```

## 2 : 

```bash
```

## 3 : 

```bash
```

## 4 : 

```bash
```

## 5 : 

```bash
```

## 6 : 

```bash
```

## 7 : 

```bash
```

## 8 : 

```bash
```

## 9 : 

```bash
```

# Cron Jobs

## 1 : 

```bash

```

## 2 : 

```bash
```

## 3 : 

```bash
```

## 4 : 

```bash
```

## 5 : 

```bash
```

## 6 : 

```bash
```

## 7 : 

```bash
```

## 8 : 

```bash
```

## 9 : 

```bash
```
