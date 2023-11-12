# Extend the Kubernetes API with CRD (CustomResourceDefinition)

## 1 : Create a CustomResourceDefinition manifest file for an Operator

with the following specifications :
- Name : operators.stable.example.com
- Group : stable.example.com
- Schema: <email: string><name: string><age: integer>
- Scope: Namespaced
- Names: <plural: operators><singular: operator><shortNames: op>
- Kind: Operator

```bash
touch files/09_01_crd.yaml
# add content
# apiVersion: apiextensions.k8s.io/v1
# kind: CustomResourceDefinition
# metadata:
#   # name must match the spec fields below, and be in the form: <plural>.<group>
#   name: operators.stable.example.com
# spec:
#   group: stable.example.com
#   versions:
#     - name: v1
#       served: true
#       # One and only one version must be marked as the storage version.
#       storage: true
#       schema:
#         openAPIV3Schema:
#           type: object
#           properties:
#             spec:
#               type: object
#               properties:
#                 email:
#                   type: string
#                 name:
#                   type: string
#                 age:
#                   type: integer
#   scope: Namespaced
#   names:
#     plural: operators
#     singular: operator
#     # kind is normally the CamelCased singular type. Your resource manifests use this.
#     kind: Operator
#     shortNames:
#     - op

```

## 2 : Create the CRD resource in the K8S API

```bash
k apply -f files/09_01_crd.yaml
```

## 3 : Create custom object from the CRD

```bash
touch files/09_02_operator.yaml
# add content
# apiVersion: stable.example.com/v1
# kind: Operator
# metadata:
#   name: operator-sample
# spec:
#   email: operator-sample@stable.example.com
#   name: "operator sample"
#   age: 30

k apply -f files/09_02_operator.yaml
```

## 4 : List operators

```bash
k get operators
k get op
```
