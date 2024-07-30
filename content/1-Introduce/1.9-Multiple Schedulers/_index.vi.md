---
title: "Multiple Schedulers"
date: "`r Sys.Date()`"
weight: 9
chapter: false
pre: "<b>  1.9 </b>"
---

Trong bài viết này, chúng ta sẽ khám phá cách triển khai nhiều **scheduler** trong một cluster **Kubernetes**. Việc có nhiều **scheduler** cho phép bạn tùy chỉnh cách các **pod** được lên lịch trên cluster của mình, tối ưu hóa việc sử dụng tài nguyên hoặc đáp ứng các yêu cầu phức tạp về cách triển khai.

#### Custom Schedulers
Một cluster **Kubernetes** có thể cấu hình để lên lịch cho các **pod** sử dụng nhiều **scheduler**. Điều này cho phép sử dụng **scheduler** mặc định của **Kubernetes** cùng với một hoặc nhiều **scheduler** tùy chỉnh.

#### Triển khai scheduler bổ sung
#### Tải xuống binary

Để thêm một **scheduler** mới, bạn cần tải xuống binary phù hợp:

```
$ wget https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-scheduler
```

#### Triển khai scheduler bổ sung - kubeadm

Sau khi có binary, bạn có thể triển khai **scheduler** mới sử dụng **kubectl**:

```
$ kubectl create -f my-custom-scheduler.yaml
```

#### Xem danh sách Schedulers
Bạn có thể xem danh sách các **scheduler** đang hoạt động trong cluster bằng cách liệt kê các **pod** trong **namespace** `kube-system`:

```
$ kubectl get pods -n kube-system
```

#### Sử dụng Custom Scheduler
Để sử dụng một **scheduler** tùy chỉnh cho một **pod** cụ thể, bạn chỉ cần thêm một trường `schedulerName` vào định nghĩa **pod** và chỉ định tên của **scheduler** bạn muốn sử dụng.

```
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

Để tạo **pod** sử dụng định nghĩa trên:

```
$ kubectl create -f pod-definition.yaml
```

Và sau đó bạn có thể kiểm tra danh sách các **pod** để xem **pod** mới đã được lên lịch chưa:

```
$ kubectl get pods
```

#### Xem Sự kiện
Bạn cũng có thể xem các sự kiện liên quan đến lên lịch **pod** để kiểm tra xem có lỗi nào không:

```
$ kubectl get events
```

#### Xem Log của Scheduler
Cuối cùng, để kiểm tra hoạt động của **scheduler** tùy chỉnh, bạn có thể xem log của nó:

```
$ kubectl logs my-custom-scheduler -n kube-system
```

#### K8s Reference Docs

Để biết thêm thông tin về cách cấu hình và sử dụng nhiều **scheduler** trong **Kubernetes**, bạn có thể tham khảo tại [tài liệu chính thức của Kubernetes](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/).
