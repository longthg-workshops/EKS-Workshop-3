---
title: "Node Affinity"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: "<b>  1.5 </b>"
---

#### Node Affinity trong Kubernetes

Node Affinity là một tính năng quan trọng trong Kubernetes, giúp đảm bảo rằng các pod được lưu trữ trên các node cụ thể, phù hợp với yêu cầu của ứng dụng. Tính năng này cho phép các nhà phát triển và quản trị hệ thống tinh chỉnh việc lên lịch chạy các pod dựa trên các thuộc tính của node, như kích thước, vùng địa lý, hoặc loại CPU.

So với Node Selectors, Node Affinity cung cấp khả năng sử dụng các biểu thức tiên tiến, cho phép xác định các quy tắc phức tạp hơn trong việc lựa chọn node cho pod.

#### Cú pháp YAML

**Ví dụ về Node Selector:**

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

**Ví dụ về Node Affinity - Sử dụng `In`:**

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

**Ví dụ về Node Affinity - Sử dụng `NotIn`:**

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

**Ví dụ về Node Affinity - Sử dụng `Exists`:**

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

#### Các Loại Node Affinity

- **requiredDuringSchedulingIgnoredDuringExecution**: Đảm bảo pod chỉ được lên lịch trên node phù hợp khi khởi tạo. Sau khi pod được khởi tạo, việc di dời pod do thay đổi thuộc tính node không được thực hiện.
- **preferredDuringSchedulingIgnoredDuringExecution**: Hệ thống sẽ cố gắng tuân theo quy tắc này nhưng không đảm bảo.

#### Kế Hoạch (Planned)

Các loại Node Affinity đang được kế hoạch hóa để triển khai trong tương lai bao gồm:
- **requiredDuringSchedulingRequiredDuringExecution**
- **preferredDuringSchedulingRequiredDuringExecution**

#### K8s Reference Docs

- [Node Affinity on Kubernetes.io](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/)
- [Advanced Scheduling in Kubernetes - Kubernetes Blog](https://kubernetes.io/blog/2017/03/advanced-scheduling-in-kubernetes/)
