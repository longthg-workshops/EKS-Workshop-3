---
title: "DaemonSets"
date: "`r Sys.Date()`"
weight: 7
chapter: false
pre: "<b>  1.7 </b>"
---

#### DaemonSets trong Kubernetes

Trong phần này, chúng ta sẽ tìm hiểu về **DaemonSets**.

**DaemonSets** giống như **ReplicaSets**, vì nó giúp triển khai nhiều phiên bản của **pod**. Nhưng **DaemonSets** chạy một bản sao của **pod** của bạn trên mỗi **node** trong **cụm** của bạn.

#### Các Trường Hợp Sử Dụng của **DaemonSets**

#### Tình huống sử dụng

**DaemonSets** thích hợp cho việc chạy một bản sao của **pod** trên mỗi **node** để thực hiện các nhiệm vụ như:

- Giám sát hệ thống
- Log collection
- Deployment của **network policy**

#### Tình huống sử dụng - Khái quát

Trong một số trường hợp, **DaemonSets** được sử dụng để triển khai các ứng dụng cần phải chạy trên tất cả hoặc một số **nodes** nhất định để cung cấp các dịch vụ cần thiết cho các **pods** khác.

#### Tình huống sử dụng - Các vấn đề cần lưu ý

Khi sử dụng **DaemonSets**, cần lưu ý đến khả năng tài nguyên và bảo mật, vì việc triển khai một **pod** trên mỗi **node** có thể ảnh hưởng đến tài nguyên hệ thống.

#### Định Nghĩa về **DaemonSets**

Tạo một **DaemonSet** tương tự như quá trình tạo **ReplicaSet**.

Đối với **DaemonSets**, chúng ta bắt đầu với `apiVersion`, `kind` là `DaemonSet` thay vì `ReplicaSet`, `metadata` và `spec`.

```yaml
apiVersion: apps/v1
kind: DaemonSet
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

Để tạo một **daemonset** từ một tệp định nghĩa:

```shell
$ kubectl create -f daemon-set-definition.yaml
```

Xem Danh Sách các **DaemonSets**
Để liệt kê các **daemonsets**:

```shell
$ kubectl get daemonsets
```

Để xem thêm thông tin chi tiết về các **daemonsets**:

```shell
$ kubectl describe daemonsets monitoring-daemon
```

#### K8s Reference Docs
[https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#writing-a-daemonset-spec](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#writing-a-daemonset-spec)
