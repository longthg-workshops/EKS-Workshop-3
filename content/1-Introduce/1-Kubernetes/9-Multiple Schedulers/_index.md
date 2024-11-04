---
title: "Multiple Schedulers"

weight: 9
chapter: false
pre: "<b> 1.1.9 </b>"
---

In this section, we will take a look at multiple schedulers in a cluster. With multiple **schedulers**, you can customize how **pods** are scheduled on your cluster, which optimizing resource utilization or meeting complex deployment requirements.

### Custom Schedulers
- Your kubernetes cluster can schedule multiple schedulers at the same time. 

  ![ms](../../images/ms.PNG)
  
### Deploy additional scheduler
#### Download the binary
  ```
  $ wget https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-scheduler
  ```
  
#### Deploy additional scheduler - kubectl
  
To create a scheduler pod

    ```
    $ kubectl create -f my-custom-scheduler.yaml
    ```
  
### View Schedulers
To list the scheduler pods

    ```
    $ kubectl get pods -n kube-system
    ```

### Use the Custom Scheduler
Create a pod definition file and add new section called **`schedulerName`** and specify the name of the new scheduler
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
  spec:
    containers:
    - image: nginx
      name: nginx
    schedulerName: my-custom-scheduler
  ```
  
To create a pod definition
  ```bash
  $ kubectl create -f pod-definition.yaml
  ```

To list pods
  ```bash
  $ kubectl get pods
  ```

## View Events

  ```bash
  $ kubectl get events
  ```
  
## View Scheduler Logs
To view scheduler logs
  ```
  $ kubectl logs my-custom-scheduler -n kube-system
  ```
  ![cs2](../../images/cs2.PNG)
  
#### K8s Reference Docs
For more information on how to configure multiple schedulers, see below:
[https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/).