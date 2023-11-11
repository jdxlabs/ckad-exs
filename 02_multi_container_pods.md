## 1 : Create a Pod with two containers, both with image busybox and command "echo hello; sleep 3600". Connect to the second container and run 'ls'

```bash
k run tmp --image=busybox $dry $o -- sh -c 'echo hello; sleep 3600' > files/02_01_po.yaml

vim files/02_01_po.yaml
# Then, copy/paste the container related values, so your final YAML should contain the following two containers (make sure those containers have a different name):

k exec -it tmp -c tmp2 -- ls
```

## 2 : Create a pod with an nginx container exposed on port 80. Add a busybox init container which downloads a page using "wget -O /work-dir/index.html http://neverssl.com/online". Make a volume of type emptyDir and mount it in both containers. For the nginx container, mount it on "/usr/share/nginx/html" and for the initcontainer, mount it on "/work-dir". When done, get the IP of the created pod and create a busybox pod and run "wget -O- IP"

```bash
k run nginx --image=nginx $dry $o --port=80 > files/02_02_po.yaml

vim files/02_02_po.yaml
# Then, add initContainers and volumes sections, so your final YAML should contain the following:
# initContainers:
# - name: init
#   image: busybox
#   command: ['sh', '-c', 'wget -O /work-dir/index.html http://neverssl.com/online']
#   volumeMounts:
#   - name: workdir
#     mountPath: /work-dir
# containers:
# - name: nginx
#   image: nginx
#   volumeMounts:
#   - name: workdir
#     mountPath: /usr/share/nginx/html
# volumes:
# - name: workdir
#   emptyDir: {}

k get po -o wide
k run tmp --image=busybox --rm -it -- sh -c 'wget -O- IP' # Replace IP with the IP of the nginx pod
```
