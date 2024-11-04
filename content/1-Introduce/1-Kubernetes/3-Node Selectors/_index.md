---
title: "Node Selectors"

weight: 3
chapter: false
pre: "<b> 1.1.3 </b>"
---

### Node Selectors in Kubernetes

**_Node Selector_** is an important feature of Kubernetes. They let you schedule certain **pods** to just run on certain **nodes** based on their pre-defined **labels**. This allows for resource optimizations and ensures that applications run on the best-fitting infrastructures.

To use **Node Selector**, add a new property into the **spec** section of the **pod** definition yaml file and define the desired labels. The **scheduler** uses these labels to match and identify the right node to place the pods on.

```yaml
[...]
spec:
  [...]S
  nodeSelector:
    size: Large
```

You also need to label your selected nodes. This can be done using the `kubectl label` command:

```bash
$ kubectl label nodes <node-name> <label-key>=<label-value>
```

For instance:

```bash
$ kubectl label nodes node-1 size=Large
```

Once you have labeled the **Node** and defined the **Node Selector** in the **Pod**, you can create the **Pod** using:

- Pod definition yaml file
```yaml
apiVersion: v1
kind: Pod
metadata:
name: myapp-pod
spec:
containers:
- name: data-processor
    image: data-processor
nodeSelector:
size: Large
```

- `kubectl create` command
```bash
$ kubectl create -f pod-definition.yml
```

### The limitations of Node Selectors

While **Node Selector** is useful, it only supports filtering based on simple labels. For more complicated requirements, Kubernetes provides **Node Affinity** and **Anti Affinity** mechanisms, which allow you to create more complex scheduling rules, including "if then" conditions and priorities.

### Reference

- [Node Selectors in Kubernetes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector)