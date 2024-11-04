---
title: "Resource Limiting"

weight: 6
chapter: false
pre: "<b> 1.1.6 </b>"
---

In this section, we will look at **Resource Limiting**.

Consider a 3-node **Kubernetes** cluster. Each node has a set of available **CPU**, **Memory**, and **Disk** resources.

If there is no sufficient resources available on any of the nodes, kubernetes holds the scheduling the pod. You will see the pod in **_pending_** state. If you look at the events, you will see the reason as insufficient CPU.

### Resource Requirements

By default, K8s assume that a pod or container within a pod requires *`0.5** CPU and **256MiB** of memory. This is known as the **Resource Request** for a container. If your application within the pod requires more than the default resources, you need to set them in the pod definition file.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
 containers:
 - name: simple-webapp-color
   image: simple-webapp-color
   ports:
    - containerPort:  8080
   resources:
     requests:
      memory: "1Gi"
      cpu: "1"
```

### Resource limiting

By default, k8s sets resource limits to 1 CPU and 512Mi of memory. You can set the resource limits for yourself in the pod definition file:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
 containers:
 - name: simple-webapp-color
   image: simple-webapp-color
   ports:
    - containerPort:  8080
   resources:
     requests:
      memory: "1Gi"
      cpu: "1"
     limits:
       memory: "2Gi"
       cpu: "2"
```

Keep in mind that **_Requests_** and **_Limits_** for resources are set **per container** in the pod.

### Resource Limit Exceeding

**_What happens when a pod tries to exceed resources beyond its limits ?_**

If a pod exceeds its resource limits, Kubernetes will take action to prevent the pod from consuming too many resources. The specific action that Kubernetes takes will depend on the type of resource that is being exceeded.

If a pod exceeds its CPU limit, Kubernetes will simply throttle the pod. This means that the pod will be given less CPU time, which can slow down the pod's performance.

On the other hand, if a pod exceeds its memory limit, Kubernetes will kill the pod with Reason OOMKilled (Out of Memory). A pod can become unresponsive or even crash when memory usage gets out of hand.

### K8s Reference Docs
[https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)