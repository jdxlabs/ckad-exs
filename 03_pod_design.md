# Labels And Annotations

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


# Pod Placement

## 13 : Create a pod that will be deployed to a Node that has the label 'accelerator=nvidia-tesla-p100'

```bash
# apply a label to a node
k label node kind1-control-plane accelerator=nvidia-tesla-p100

# show labels of nodes
k get nodes --show-labels

k run nginx --image=nginx --restart=Never --labels=accelerator=nvidia-tesla-p100
# it runs even if the node isn't labeled
```

## 14 : Taint a node with key tier and value frontend with the effect NoSchedule. Then, create a pod that tolerates this taint.

```bash
k taint node kind1-control-plane tier=frontend:NoSchedule

# show taints of nodes
k get nodes -o json | jq '.items[].spec.taints'

k run nginx --image=nginx $dry $o > files/03_14_po.yml

# add toleration to nginx.yaml, after the containers section
tolerations:
- key: "tier"
  operator: "Equal"
  value: "frontend"
  effect: "NoSchedule"

k apply -f files/03_14_po.yml
```

## 15 : Create a pod that will be placed on node kind1-control-plane. Use nodeSelector and tolerations.

```bash
k run nginx --image=nginx $dry $o > files/03_15_po.yml

# Add nodeSelector and tolerations in the "spec" section
  nodeSelector:
    kubernetes.io/hostname: kind1-control-plane
  tolerations:
  - key: "node-role.kubernetes.io/control-plane"
    operator: "Exists"
    effect: "NoSchedule"
```


# Deployments

## 16 : Create a deployment with image nginx:1.18.0, called nginx, having 2 replicas, defining port 80 as the port that this container exposes (don't create a service for this deployment)

```bash
k create deploy nginx --image=nginx:1.18.0 --replicas=2 --port=80
```

## 17 : View the YAML of this deployment

```bash
k get deploy nginx $o
```

## 18 : View the YAML of the replica set that was created by this deployment

```bash
k get rs nginx-54bcfc567b $o
```

## 19 : Get the YAML for one of the pods

```bash
k get po nginx-54bcfc567b-c8g7m $o
```

## 20 : Check how the deployment rollout is going

```bash
k rollout status deploy nginx
```

## 21 : Update the nginx image to nginx:1.19.8

```bash
k set image deploy nginx nginx=nginx:1.19.8
# or k edit deploy..
```

## 22 : Check the rollout history and confirm that the replicas are OK

```bash
k rollout history deploy nginx
k get rs  # 2 newly created replicas
```

## 23 : Undo the latest rollout and verify that new pods have the old image (nginx:1.18.0)

```bash
k rollout undo deploy nginx
k describe po nginx-54bcfc567b-h8v86  # check that the old image has been applied
```

## 24 : Do an on purpose update of the deployment with a wrong image nginx:1.91

```bash
k set image deploy nginx nginx=nginx:1.91
# or k edit deploy..
```

## 25 : Verify that something's wrong with the rollout

```bash
k rollout status deploy nginx
# or k get po => ImagePullBackOff
```

## 26 : Return the deployment to the second revision (number 2) and verify the image is nginx:1.19.8

```bash
k rollout undo deploy nginx --to-revision=2
k rollout status deploy nginx
k describe po | grep -i image:
```

## 27 : Check the details of the fourth revision (number 4)

```bash
k rollout history deploy nginx --revision=4
```

## 28 : Scale the deployment to 5 replicas

```bash
k scale deploy nginx --replicas=5
```

## 29 : Autoscale the deployment, pods between 5 and 10, targetting CPU utilization at 80%

```bash
k autoscale deploy nginx --min=5 --max=10 --cpu-percent=80
k get hpa
```

## 30 : Pause the rollout of the deployment

```bash
k rollout pause deploy nginx
```

## 31 : Update the image to nginx:1.19.9 and check that there's nothing going on, since we paused the rollout

```bash
k set image deploy nginx nginx=nginx:1.19.9
k rollout status deploy nginx
```

## 32 : Resume the rollout and check that the nginx:1.19.9 image has been applied

```bash
k rollout resume deploy nginx
k rollout status deploy nginx
k describe po | grep -i image:
```

## 33 : Delete the deployment and the horizontal pod autoscaler you created

```bash
k delete deploy nginx
k delete hpa nginx
```

## 34 : Implement canary deployment by running two instances of nginx marked as version=v1 and version=v2 so that the load is balanced at 75%-25% ratio

```bash
# k create deploy nginx --image=nginx --labels=version=v1 $dry $o > files/03_34_deploy1_po.yml
# k create deploy nginx --image=nginx --labels=version=v2 $dry $o > files/03_34_deploy2_po.yml
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
