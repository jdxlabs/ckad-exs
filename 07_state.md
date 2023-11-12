# State Persistence - Define volumes

## 1 : Create busybox pod with two containers, each one will have the image busybox and will run the 'sleep 3600' command. Make both containers mount an emptyDir at '/etc/foo'. Connect to the second busybox, write the first column of '/etc/passwd' file to '/etc/foo/passwd'. Connect to the first busybox and write '/etc/foo/passwd' file to standard output. Delete pod.

```bash
k run busybox --image=busybox $dry $o -- sh -c "sleep 3600" > files/07_01_po.yaml

# add the following
# volumes:
# - name: myvolume
#   emptyDir: {}
# containers:
# - name: busybox
#   image: busybox
#   args:
#   - sh
#   - -c
#   - sleep 3600
#   resources: {}
#   volumeMounts:
#   - mountPath: /etc/foo
#     name: myvolume
# - name: busybox2
#   image: busybox
#   args:
#   - sh
#   - -c
#   - sleep 3600
#   resources: {}
#   volumeMounts:
#   - mountPath: /etc/foo
#     name: myvolume

k exec -it busybox -c busybox2 -- sh
cat /etc/passwd | cut -f 1 -d ':' > /etc/foo/passwd

k exec -it busybox -c busybox -- sh
```

## 2 : Create a PersistentVolume of 10Gi, called 'myvolume'. Make it have accessMode of 'ReadWriteOnce' and 'ReadWriteMany', storageClassName 'normal', mounted on hostPath '/etc/foo'. Save it on pv.yaml, add it to the cluster. Show the PersistentVolumes that exist on the cluster

```bash
touch files/07_02_pv.yaml
# add the following
# apiVersion: v1
# kind: PersistentVolume
# metadata:
#   name: myvolume
# spec:
#   capacity:
#     storage: 10Gi
#   accessModes:
#   - ReadWriteOnce
#   - ReadWriteMany
#   hostPath:
#     path: /etc/foo
#   storageClassName: normal
```

## 3 : Create a PersistentVolumeClaim for this storage class, called 'mypvc', a request of 4Gi and an accessMode of ReadWriteOnce, with the storageClassName of normal, and save it on pvc.yaml. Create it on the cluster. Show the PersistentVolumeClaims of the cluster. Show the PersistentVolumes of the cluster

```bash
touch files/07_03_pvc.yaml
# add the following
# apiVersion: v1
# kind: PersistentVolumeClaim
# metadata:
#   name: mypvc
# spec:
#   accessModes:
#   - ReadWriteOnce
#   resources:
#     requests:
#       storage: 4Gi
#   storageClassName: normal
```

## 4 : Create a busybox pod with command 'sleep 3600', save it on pod.yaml. Mount the PersistentVolumeClaim to '/etc/foo'. Connect to the 'busybox' pod, and copy the '/etc/passwd' file to '/etc/foo/passwd'

```bash
k run busybox --image=busybox $dry $o -- sh -c "sleep 3600" > files/07_04_po.yaml
# add the following
# volumes:
# - name: myvolume
#   persistentVolumeClaim:
#     claimName: mypvc
# containers:
# - name: busybox
#   image: busybox
#   args:
#   - sh
#   - -c
#   - sleep 3600
#   resources: {}
#   volumeMounts:
#   - mountPath: /etc/foo
#     name: myvolume

k exec -it busybox -- sh
# cp /etc/passwd /etc/foo/passwd
# cat /etc/foo/passwd
```

## 5 : Create a second pod which is identical with the one you just created (you can easily do it by changing the 'name' property on pod.yaml). Connect to it and verify that '/etc/foo' contains the 'passwd' file. Delete pods to cleanup. Note: If you can't see the file from the second pod, can you figure out why? What would you do to fix that?

```bash
cp files/07_04_po.yaml files/07_05_po.yaml
# change the pod name to busybox2

k exec -it busybox2 -- sh
# cat /etc/foo/passwd

k delete po --all $f
```

## 6 : Create a busybox pod with 'sleep 3600' as arguments. Copy '/etc/passwd' from the pod to your local folder

```bash
k run busybox --image=busybox -- sh -c "sleep 3600"

k cp busybox:/etc/passwd ./passwd
rm passwd
```
