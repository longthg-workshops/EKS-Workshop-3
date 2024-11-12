---
title: "Labels and Selectors"

weight: 2
chapter: false
pre: "<b> 1.1.2 </b>"
---

In this section, we will take a look at **`Labels and Selectors`**

**Labels** and **Selectors** are standard methods to group things together. **Labels** are properties attached to each item, while **Selectors** help you to filter these items.
  
### How are labels and selectors are used in kubernetes?

- We have created different types of objects in kubernetes such as **`PODs`**, **`ReplicaSets`**, **`Deployments`** etc.
  
The question, is **_"How do you specify labels ?"_** It is quite simple. You just need to create one in the Metadata of the yaml definition file:

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: simple-webapp
    labels:
    app: App1
    function: Front-end
spec:
    containers:
    - name: simple-webapp
    image: simple-webapp
    ports:
    - containerPort: 8080
```
 
Once the pod is created, to select the pod with labels run the below command
```bash
$ kubectl get pods --selector app=App1
```

Kubernetes uses labels to connect different objects together. For example, you can create a StatefulSet of pods with the following yaml file:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
    name: simple-webapp
    labels:
    app: App1
    function: Front-end
spec:
    replicas: 3
    selector:
    matchLabels:
    app: App1
    template:
    metadata:
        labels:
        app: App1
        function: Front-end
    spec:
        containers:
        - name: simple-webapp
        image: simple-webapp   
```

To create Service that routes requests to the pods inside the ReplicaSet above, we create the following YAML:
 
```yaml
apiVersion: v1
kind: Service
metadata:
name: my-service
spec:
selector:
    app: App1
ports:
- protocol: TCP
    port: 80
    targetPort: 9376 
```
  
## Annotations
- While labels and selectors are used to group objects, annotations are used to record other details for informative purpose.
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
    name: simple-webapp
    labels:
    app: App1
    function: Front-end
    annotations:
        buildversion: 1.34
spec:
    replicas: 3
    selector:
    matchLabels:
    app: App1
template:
    metadata:
    labels:
        app: App1
        function: Front-end
    spec:
    containers:
    - name: simple-webapp
        image: simple-webapp   
```

### K8s Reference Docs:
- https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/