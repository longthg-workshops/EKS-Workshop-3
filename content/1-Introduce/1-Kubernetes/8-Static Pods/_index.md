---
title: "Static Pods"
weight: 8
chapter: false
pre: "<b> 1.1.8 </b>"
---

In this section, we will take a look at Static Pods

#### How do you provide a pod definition file to the kubelet without a kube-apiserver?
You can configure the kubelet to read the pod definition files from a directory on the server designated to store information about pods.

### Configure Static Pod
The designated directory can be any directory on the host and the location of that directory is passed in to the kubelet as an option while running the service.

The option is named as **`--pod-manifest-path`**.
  
### Another way to configure static pod 
Instead of specifying the option directly in the **`kubelet.service`** file, you could provide a path to another config file using the config option, and define the directory path as staticPodPath in the file.

### View the static pods
To view the static pods

  ```bash
  $ docker ps
  ```


The ***kubelet*** can create both kinds of pods - the static pods and the ones from the api server at the same time.


### Static Pods - Use Cases

Static Pods in Kubernetes are pods that are managed directly by the kubelet daemon on a specific node, without going through the apiserver. This means that they cannot be managed or controlled remotely by other parts of Kubernetes such as `kubectl` or the apiserver. Static Pods are often used in specific cases, such as launching Kubernetes system components such as kube-proxy, etcd, or the master kube-apiserver on the cluster.

#### 1. **Use Case: Managing Kubernetes Clusters in Environments with No or Limited Network Access**

When deploying Kubernetes in an environment with no or limited network access (e.g., private networks, offline production environments), using Static Pods allows administrators to deploy and manage core Kubernetes components without requiring network connectivity to the apiserver. This ensures that the cluster can continue to operate even if the network connection is lost.

#### 2. **Use Case: Bootstrapping a Kubernetes cluster**

During the initialization of a Kubernetes cluster, Static Pods are often used to run core cluster components such as kube-apiserver, etcd, and controller manager before the cluster can manage itself via the apiserver. This helps to automatically initialize the cluster without manual intervention.

#### 3. **Use Case: Running applications that require high availability**

Since Static Pods are managed directly by kubelet, they will be automatically restarted by kubelet if they fail. This can be useful for applications that need high availability and self-healing without relying on the usual Kubernetes control mechanisms.

#### 4. **Use Case: Simple Deployment in Resource-Constrained Environments**

In some cases, deploying a full Kubernetes cluster may not be feasible due to system resource constraints. In these cases, using Static Pods allows applications to be deployed simply without complex control components, reducing resource requirements.
  
### Static Pods vs DaemonSets

| Criteria | Static Pod | DaemonSets |
|-------------------|-------------------------- ---------|--------------------------------------- -|
| **Initialization** | Initialized by `kubelet` on each node | Initialized by API server via DaemonSet description |
| **Management** | Directly managed by `kubelet` on the node on which it runs | Managed via API server and other control mechanisms |
| **Update** | Update by modifying the config file on the node(s) | Update information via DaemonSet description update in API server |
| **Scope of operation** | Only runs on a specific node, not managed by API server | Runs on all nodes or selected nodes via labels |
| **Intended use case** | Suitable for low-level and base system services such as kube-proxy, kube-dns | Suitable for services that need to run on each node, such as log collectors, monitoring agents |
| **Automatic restart** | Kubelet will automatically restart the Pod if it fails | API server ensures that the Pod is always running on the nodes as described |
| **Flexibility** | Less flexible as it requires modifying the file configuration and managing it directly on each node | More flexible, can be easily updated and managed from one place API server via hub |
| **Manageability** | More difficult to centralize management, requiring direct intervention on each node | Easy to centrally manage your information via kubectl and other tools via API server |

#### K8s Reference Docs
- https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/