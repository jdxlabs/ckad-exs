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

# and for all pods
k describe po $o | grep -i -A5 metadata
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
k create deploy nginx --image=nginx $dry $o > files/03_34_deploy1_po.yml
# add label : version: v1
# and an initContainer to show version
#   volumeMounts:
#   - name: workdir
#     mountPath: /usr/share/nginx/html
# initContainers:
# - name: install
#   image: busybox:1.28
#   command:
#   - /bin/sh
#   - -c
#   - "echo version-1 > /work-dir/index.html"
#   volumeMounts:
#   - name: workdir
#     mountPath: "/work-dir"
# volumes:
# - name: workdir
#   emptyDir: {}

k create deploy nginx --image=nginx $dry $o > files/03_34_deploy2_po.yml
# add label : version: v2
# and an initContainer to show version

k create service clusterip nginx --tcp=80:80 $dry $o > files/03_34_svc_po.yml

k apply -f files/03_34_deploy1_po.yml
k apply -f files/03_34_deploy2_po.yml
k apply -f files/03_34_svc.yml

# check the response, at 75-25 ratio :
k run tmp --image=busybox --rm -it --restart=Never -- sh -c 'while sleep 1; do wget -qO- http://nginx-svc; done'
```


# Jobs

## 35 : Create a job named pi with image perl:5.34 that runs the command with arguments "perl -Mbignum=bpi -wle 'print bpi(2000)'"

```bash
k create job pi --image=perl:5.34 -- sh -c "perl -Mbignum=bpi -wle 'print bpi(2000)'"
```

## 36 : Wait till it's done, get the output

```bash
k get po -w
k logs pi-psf26
```

## 37 : Create a job with the image busybox that executes the command 'echo hello;sleep 30;echo world'

```bash
k create job box --image=busybox -- sh -c "echo hello;sleep 30;echo world"
```

## 38 : Follow the logs for the pod (you'll wait for 30 seconds)

```bash
k logs -f box-q94h7
```

## 39 : See the status of the job, describe it and see the logs

```bash
k get po box-q94h7
k describe po box-q94h7
k logs box-q94h7
```

## 40 : Delete the job

```bash
k delete job box
```

## 41 : Create a job but ensure that it will be automatically terminated by kubernetes if it takes more than 30 seconds to execute

```bash
k create job box --image=busybox $dry $o -- sh -c "echo hello;sleep 30;echo world" > files/03_41_job.yml

# add the spec : 
# spec:
#   activeDeadlineSeconds: 30
```

## 42 : Create the same job, make it run 5 times, one after the other. Verify its status and delete it

```bash
k create job box --image=busybox $dry $o -- sh -c "echo hello;sleep 30;echo world" > files/03_42_job.yml

# add the spec :
# spec:
#   completions: 5
```

## 43 : Create the same job, but make it run 5 parallel times

```bash
k create job box --image=busybox $dry $o -- sh -c "echo hello;sleep 30;echo world" > files/03_43_job.yml

# add the spec :
# spec:
#   parallelism: 5
```


# Cron Jobs

## 44 : Create a cron job with image busybox that runs on a schedule of "*/1 * * * *" and writes 'date; echo Hello from the Kubernetes cluster' to standard output

```bash
k create cronjob box --image=busybox --schedule="*/1 * * * *" -- sh -c "date; echo Hello from the Kubernetes cluster"
```

## 45 : See its logs and delete it

```bash
k logs box-28316487-sg7kl
k delete cronjob box
```

## 46 : Create the same cron job again, and watch the status. Once it ran, check which job ran by the created cron job. Check the log, and delete the cron job

```bash
k create cronjob box --image=busybox --schedule="*/1 * * * *" -- sh -c "date; echo Hello from the Kubernetes cluster"
k get cj,job,po
k logs box-28316490-fbh6f
k delete cj box
```

## 47 : Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it takes more than 17 seconds to start execution after its scheduled time (i.e. the job missed its scheduled time) 

```bash
k create cronjob box --image=busybox --schedule="*/1 * * * *" $dry $o -- sh -c "date; echo Hello from the Kubernetes cluster" > files/03_47_cronjob.yml

# add the spec :
# spec:
#   startingDeadlineSeconds: 17
```

## 48 : Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it successfully starts but takes more than 12 seconds to complete execution

```bash
k create cronjob box --image=busybox --schedule="*/1 * * * *" $dry $o -- sh -c "date; echo Hello from the Kubernetes cluster" > files/03_48_cronjob.yml

# add the spec (in jobTemplate) :
# spec:
#   activeDeadlineSeconds: 12
```

## 49 : Create a job from cronjob

```bash
k create job --from=cronjob/box new-job
```
