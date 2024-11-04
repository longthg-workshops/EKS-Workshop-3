---
title: "Node Affinity"

weight: 5
chapter: false
pre: "<b> 1.1.5 </b>"
---

Node Affinity is an important feature in Kubernetes that ensures that pods are hosted on specific nodes, matching the requirements of the application. This feature allows developers and system administrators to fine-tune the scheduling of pods based on node properties - such as size, geographical location, or CPU type.

In comparison to Node Selectors, Node Affinity provides the ability to use advanced expressions, allowing for more complex rules to be defined when selecting nodes for pods.

### YAML syntax

**Node Selector:**

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

**Node Affinity - Using the `In` operator:**

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
 containers:
 - name: data-processor
   image: data-processor
 affinity:
   nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
       - matchExpressions:
         - key: size
           operator: In
           values: 
           - Large
           - Medium
```

**Node Affinity - Using `NotIn`:**

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
 containers:
 - name: data-processor
   image: data-processor
 affinity:
   nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
       - matchExpressions:
         - key: size
           operator: NotIn
           values: 
           - Small
```

**Node Affinity - Using `Exists`:**

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
 containers:
 - name: data-processor
   image: data-processor
 affinity:
   nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
       - matchExpressions:
         - key: size
           operator: Exists
```

### Node Affinity Types
#### Available

  - **requiredDuringSchedulingIgnoredDuringExecution**: Ensures pods are only scheduled on the appropriate node upon initialization. Once a pod is initialized, pod relocation due to node attribute changes is not performed.
  - **preferredDuringSchedulingIgnoredDuringExecution**: The system will attempt to follow this rule but does not guarantee it.

#### Planned
  - **requiredDuringSchedulingRequiredDuringExecution**
  - **preferredDuringSchedulingRequiredDuringExecution**

### References

- [Node Affinity on Kubernetes.io](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/)
- [Advanced Scheduling in Kubernetes - Kubernetes Blog](https://kubernetes.io/blog/2017/03/advanced-scheduling-in-kubernetes/)
- [Node Affinity - requiredDuringSchedulingRequiredDuringExecution](https://github.com/kubernetes/kubernetes/issues/96149)