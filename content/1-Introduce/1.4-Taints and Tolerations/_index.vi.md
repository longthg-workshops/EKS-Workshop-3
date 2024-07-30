---
title: "Taints and Tolerations"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b>  1.4 </b>"
---

Trong phần này, chúng ta sẽ tìm hiểu về **taints** và **tolerations**.

- Mối quan hệ giữa **Pod** và **node** và cách bạn có thể hạn chế **Pod** được đặt trên node nào.
- **Taints** và **Tolerations** được sử dụng để đặt các hạn chế về việc **Pod** nào có thể được lên lịch trên một **node**.
- Chỉ có các **Pod** chịu đựng được **taint** cụ thể trên một **node** mới có thể được lên lịch trên **node** đó.

#### Taints
- Sử dụng lệnh `kubectl taint nodes` để tạo một **taint** trên một **node**.

- Cú pháp

```
$ kubectl taint nodes <tên-node> key=value:taint-effect

```

- Ví dụ

```
$ kubectl taint nodes node1 app=blue:NoSchedule

```

- **Taint effect** xác định điều gì sẽ xảy ra với các **Pod** nếu chúng không chịu đựng được **taint**.

- Có 3 tác động **taint**:

  - **NoSchedule**
  - **PreferNoSchedule**
  - **NoExecute**

#### Tolerations
- **Tolerations** được thêm vào các **Pod** bằng cách thêm một phần **tolerations** trong định nghĩa của **Pod**.

```
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

- **Taints** và **Tolerations** không chỉ định cho **Pod** đi tới một **node** cụ thể. Thay vào đó, chúng chỉ định cho **node** chỉ chấp nhận các **Pod** có các **toleration** cụ thể.
- Để xem **taint** này, chạy lệnh dưới đây:

```
$ kubectl describe node kubemaster |grep Taint

```

#### Tài liệu tham khảo Kubernetes
- [Kubernetes Official Documentation on Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
