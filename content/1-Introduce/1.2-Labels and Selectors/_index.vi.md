---
title: "Labels and Selectors"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 1.2 </b>"
---

#### Nhãn và Bộ chọn

Nhãn và Bộ chọn là các phương pháp tiêu chuẩn trong **AWS**, **Kubernetes** và ngành **IT** để nhóm các đối tượng cùng nhau. **Nhãn** là các thuộc tính được gắn vào mỗi mục, trong khi **Bộ chọn** giúp bạn lọc các mục này theo nhãn.

**Làm thế nào để sử dụng nhãn và bộ chọn trong Kubernetes?**

Trong **Kubernetes**, chúng ta có thể tạo ra các loại đối tượng khác nhau như **PODs**, **ReplicaSets**, **Deployments** và nhiều hơn nữa. Nhãn giúp quản lý và lọc các đối tượng này một cách linh hoạt.

**Làm thế nào để xác định các nhãn?**

Ví dụ, để xác định một **Pod** với nhãn, cấu hình như sau:

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

Sau khi **Pod** được tạo, để chọn **Pod** với nhãn, chạy lệnh:

```shell
$ kubectl get pods --selector app=App1
```

**Kubernetes** sử dụng nhãn để kết nối các đối tượng khác nhau, ví dụ như trong một **ReplicaSet**:

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

- Đối với **Dịch vụ**:

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

#### Chú thích

Trong khi **nhãn** và **bộ chọn** được sử dụng để nhóm các đối tượng, **chú thích** được sử dụng để ghi lại các chi tiết khác cho mục đích thông tin.

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

#### Tài liệu Tham khảo K8s:

- [Kubernetes: Working with Objects - Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)