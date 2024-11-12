---
title: "DaemonSets"
weight: 7
chapter: false
pre: "<b> 1.1.7 </b>"
---

In this section, we will take a look at DaemonSets.

DaemonSets are like ReplicaSets. They helps in to deploy multiple instances of pod. But it runs one copy of your pod on each node in your cluster.
  
### Use Cases
**DaemonSets** are suitable for running a copy of a **pod** on each **node** to perform tasks such as:

- System monitoring
- Log collection
- **Network policy** deployment

In certain cases, **DaemonSets** are used to deploy applications that need to run on all or certain **nodes** to provide necessary services to other **pods**.

{{% notice note %}}
When using **DaemonSets**, resource and security considerations should be taken into account, as deploying one **pod** per **node** can impact system resources.
{{% /notice %}}
  
### Creating a DaemonSet

Creating a DaemonSet is similar to the ReplicaSet creation process.

For DaemonSets, we start with apiVersion, kind as **`DaemonSets`** instead of **`ReplicaSet`**, metadata and spec.

```yaml
apiVersion: apps/v1
kind: Replicaset
metadata:
name: monitoring-daemon
labels:
    app: nginx
spec:
selector:
    matchLabels:
    app: monitoring-agent
template:
    metadata:
    labels:
        app: monitoring-agent
    spec:
    containers:
    - name: monitoring-agent
      image: monitoring-agent
```
  
To create a daemonset from a definition file:

```bash
$ kubectl create -f daemon-set-definition.yaml
```

### View DaemonSets

To list daemonsets

```bash
$ kubectl get daemonsets
```

For more details of the daemonsets

```bash
$ kubectl describe daemonsets monitoring-daemon
```
  
### K8s Reference Docs
- https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#writing-a-daemonset-spec