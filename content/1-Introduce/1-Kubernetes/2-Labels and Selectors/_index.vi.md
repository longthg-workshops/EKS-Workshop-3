---
title: "Labels and Selectors"

weight: 2
chapter: false
pre: "<b> 1.1.2 </b>"
---

#### Label và Selector

Label và Selector là các phương pháp tiêu chuẩn trong **AWS**, **Kubernetes** và ngành **IT** để nhóm các đối tượng cùng nhau. **Label** là các thuộc tính được gắn vào mỗi mục, trong khi **Selector** giúp bạn lọc các mục này theo Label.

**Làm thế nào để sử dụng Label và Selector trong Kubernetes?**

Trong **Kubernetes**, chúng ta có thể tạo ra các loại đối tượng khác nhau như **PODs**, **ReplicaSets**, **Deployments** và nhiều hơn nữa. Label giúp quản lý và lọc các đối tượng này một cách linh hoạt.

**Làm thế nào để xác định các Label?**

Ví dụ, để xác định một **Pod** với Label, cấu hình như sau:

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

Sau khi **Pod** được tạo, để chọn **Pod** với Label, chạy lệnh:

```shell
$ kubectl get pods --selector app=App1
```

**Kubernetes** sử dụng Label để kết nối các đối tượng khác nhau, ví dụ như trong một **ReplicaSet**:

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

Trong khi **Label** và **Selector** được sử dụng để nhóm các đối tượng, **chú thích** được sử dụng để ghi lại các chi tiết khác cho mục đích thông tin.

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