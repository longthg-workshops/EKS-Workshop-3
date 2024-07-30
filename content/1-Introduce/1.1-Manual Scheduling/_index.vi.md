---
title: "Manual Scheduling"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 1.1 </b>"
---

#### Cách thức Lập lịch hoạt động

Bạn làm gì khi bạn không có một **scheduler** trong cluster của bạn?

- Mỗi **POD** có một trường gọi là **NodeName** mà theo mặc định không được đặt. Bạn thường không chỉ định trường này khi bạn tạo tệp biểu thị, **Kubernetes** tự động thêm nó.
- Khi xác định được nó, **Kubernetes** lập lịch cho **POD** trên node bằng cách thiết lập thuộc tính **nodeName** thành tên của node bằng cách tạo một đối tượng liên kết.

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: nginx
 labels:
  name: nginx
spec:
 containers:
 - name: nginx
   image: nginx
   ports:
   - containerPort: 8080
 nodeName: node02
```

### Không có Scheduler

Bạn có thể gán thủ công các **pod** cho node chính nó. Và không có một **scheduler**, để lập lịch cho **pod** là thiết lập thuộc tính **nodeName** trong tệp định nghĩa **pod** của bạn khi tạo một **pod**.

#### Cách khác

```yaml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02
---
apiVersion: v1
kind: Pod
metadata:
 name: nginx
 labels:
  name: nginx
spec:
 containers:
 - name: nginx
   image: nginx
   ports:
   - containerPort: 8080
```

#### K8s Reference Docs:

- [Kubernetes API Concepts](https://kubernetes.io/docs/reference/using-api/api-concepts/)
- [Assign POD to Node](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodename)
