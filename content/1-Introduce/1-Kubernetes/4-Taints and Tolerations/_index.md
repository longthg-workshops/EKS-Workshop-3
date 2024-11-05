---
title: "Taints and Tolerations"

weight: 4
chapter: false
pre: "<b> 1.1.4 </b>"
---

In this section, we will learn about **taints** and **tolerations**.

**Taints** and **Tolerations** affect the relationship between **pods** and **nodes** and how you can restrict **pods** from being placed on specific nodes. **Taints** and **Tolerations** are used to place restrictions on which **Pods** can be scheduled to a **node**. Only **Pods** that have **tolerations** for specific **taints** (taints tolerant) on a **node** can be scheduled on that **node**.

### Taints
Use **`kubectl taint nodes`** command to taint a node.

- Syntax:

  ```bash
  $ kubectl taint nodes <node-name> key=value:taint-effect
  ```
 
- Example:

  ```bash
  $ kubectl taint nodes node1 app=blue:NoSchedule
  ```
  
- The taint effect defines what would happen to the pods if they do not tolerate the taint.
- There are 3 taint effects
  - **`NoSchedule`**
  - **`PreferNoSchedule`**
  - **`NoExecute`**
  
### Tolerations
- Tolerations are added to pods by adding a **`tolerations`** section in pod definition.

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
        name: myapp-pod
        spec:
            containers:
                - name: nginx-container
                image: nginx
            tolerations:
                - key: "app"
                operator: "Equal"
                value: "blue"
                effect: "NoSchedule"
    ```
    
### Reference
- [Kubernetes Official Documentation on Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)