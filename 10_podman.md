# Podman basics - Define, build and modify container images

## 1 : Create a Dockerfile to deploy an Apache HTTP Server which hosts a custom main page

```bash
mkdir files/podman
touch files/podman/Dockerfile

# add content
# FROM docker.io/httpd:2.4
# RUN echo "Hello, World!" > /usr/local/apache2/htdocs/index.html

```

## 2 : Build and see how many layers the image consists of

```bash
cd files/podman
podman build -t simpleapp .

podman images

podman image tree simpleapp:latest
```

## 3 : Run the image locally, inspect its status and logs, finally test that it responds as expected

```bash
podman run -d --name test -p 8080:80 simpleapp

podman inspect simpleapp

podman ps
podman logs CONTAINER_ID

curl http://localhost:8080
```

## 4 : Run a command inside the pod to print out the index.html file

```bash
podman exec -it test sh
# cat /usr/local/apache2/htdocs/index.html
```

## 5 : Tag the image with ip and port of a private local registry and then push the image to this registry

```bash
podman tag simpleapp $registry_ip:5000/simpleapp
podman push $registry_ip:5000/simpleapp
```

## 6 : Verify that the registry contains the pushed image and that you can pull it

```bash
curl http://$registry_ip:5000/v2/_catalog

podman pull $registry_ip:5000/simpleapp
```

## 7 : Run a pod with the image pushed to the registry

```bash
podman run -d --name test -p 8080:80 $registry_ip:5000/simpleapp
```

## 8 : Log into a remote registry server and then read the credentials from the default file

```bash
podman login --username $YOUR_USER --password $YOUR_PWD docker.io

cat ${XDG_RUNTIME_DIR}/containers/auth.json
# {
#         "auths": {
#                 "docker.io": {
#                         "auth": "Z2l1bGl0JLSGtvbkxCcX1xb617251xh0x3zaUd4QW45Q3JuV3RDOTc="
#                 }
#         }
# }
```

## 9 : Create a secret both from existing login credentials and from the CLI

```bash
k create secret generic mysecret --from-file=.dockerconfigjson=${XDG_RUNTIME_DIR}/containers/auth.json --type=kubernetes.io/dockeconfigjson

k create secret docker-registry mysecret2 --docker-server=https://index.docker.io/v1/ --docker-username=$YOUR_USR --docker-password=$YOUR_PWD
```

## 10 : Create the manifest for a Pod that uses one of the two secrets just created to pull an image hosted on the relative private remote registry

```bash
# apiVersion: v1
# kind: Pod
# metadata:
#   name: private-reg
# spec:
#   containers:
#   - name: private-reg-container
#     image: $YOUR_PRIVATE_IMAGE
#   imagePullSecrets:
#   - name: mysecret
```

## 11 : Clean up all the images and containers

```bash
podman rm --all --force
podman rmi --all
k delete pod simpleapp
```
