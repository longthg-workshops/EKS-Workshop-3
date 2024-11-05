---
title: "Taints and Tolerations"

weight: 4
chapter: false
pre: "<b> 1.1.4 </b>"
---

Trong phần này, chúng ta sẽ tìm hiểu về **taints** và **tolerations**.

**Taints** và **Tolerations** ảnh hưởng đến mối quan hệ giữa **pod** và **node** và cách bạn có thể hạn chế **pod** được đặt trên node cụ thể. **Taints** và **Tolerations** được sử dụng để đặt các hạn chế về việc **Pod** nào có thể được lên lịch cho một **node**. Chỉ các **Pod** được gắn **toleration** cho các **taint** cụ thể trên một **node** mới có thể được lên lịch trên **node** đó.

### Taints
- Sử dụng lệnh `kubectl taint nodes` để tạo một **taint** trên một **node**.

- Cú pháp

  ```bash
  $ kubectl taint nodes <tên-node> key=value:taint-effect
  ```

- Ví dụ

  ```bash
  $ kubectl taint nodes node1 app=blue:NoSchedule
  ```

- **Taint effect** xác định điều gì sẽ xảy ra với các **Pod** nếu chúng không được gắn **toleration** cho **taint**.

- Có 3 kiểu tác động của **taint**:
  - **NoSchedule**
  - **PreferNoSchedule**
  - **NoExecute**

### Tolerations
- **Tolerations** được thêm vào các **Pod** bằng cách thêm một phần **tolerations** trong định nghĩa của **Pod**.

```yaml
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

**Taints** và **Tolerations** không chỉ định cho **Pod** đi tới một **node** cụ thể. Thay vào đó, chúng chỉ định cho **node** chỉ chấp nhận các **Pod** có các **toleration** cụ thể.

Để xem **taint** này, chạy lệnh dưới đây:

```bash
$ kubectl describe node kubemaster |grep Taint
```

### Tài liệu tham khảo Kubernetes
- [Kubernetes Official Documentation on Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
