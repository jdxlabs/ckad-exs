# Managing Kubernetes with Helm - Helm in K8s

## 1 : Creating a basic Helm chart

```bash
helm create files/helmchart
```

## 2 : Running a Helm chart

```bash
# edit files/helmchart/values.yaml
# replicaCount: 2
# image:
#   repository: nginx
#   tag: stable
helm install demo-release files/helmchart
```

## 3 : Find pending Helm deployments on all namespaces

```bash
helm ls --pending -A
```

## 4 : Uninstall a Helm release

```bash
helm uninstall demo-release
```

## 5 : Upgrading a Helm chart

```bash
helm upgrade demo-release files/helmchart --set replicaCount=3
```

## 6 : Using Helm repo

```bash
# helm repo add [NAME] [URL]  [flags]

# helm repo ls

# helm repo remove [REPO1] [flags]

# helm repo update [REPO1] [flags]

# helm repo index [DIR] [flags]
```

## 7 : Download a Helm chart from a repository

```bash
# helm pull [chart URL | repo/chartname] [...] [flags] ## this would download a helm, not install 
# helm pull --untar [rep/chartname] # untar the chart after downloading it 
```

## 8 : Add the Bitnami repo at https://charts.bitnami.com/bitnami to Helm

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

## 9 : Write the contents of the values.yaml file of the bitnami/node chart to standard output

```bash
helm show values bitnami/node
```

## 10 : Install the bitnami/node chart setting the number of replicas to 5

```bash
helm install node bitnami/node --set replicaCount=5
helm uninstall node
```
