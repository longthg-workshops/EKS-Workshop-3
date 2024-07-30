---
title: "Node Selectors"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b>  1.3 </b>"
---

#### Node Selectors trong Kubernetes

Node Selectors là một tính năng quan trọng trong Kubernetes, cho phép bạn hạn chế pod chỉ được lên lịch chạy trên một nhóm các **Node** cụ thể dựa trên các nhãn (**labels**) được chỉ định. Điều này giúp tối ưu hóa việc sử dụng tài nguyên và đảm bảo rằng các ứng dụng chạy trên cơ sở hạ tầng phù hợp nhất.

- Để sử dụng Node Selector, bạn cần thêm một thuộc tính mới vào phần **spec** của định nghĩa **Pod** và chỉ định nhãn phù hợp.

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

- Bạn cũng cần gán nhãn cho các **Node** mà bạn muốn chọn. Điều này có thể được thực hiện sử dụng lệnh `kubectl label`.

```bash
$ kubectl label nodes <node-name> <label-key>=<label-value>
```

Ví dụ:

```bash
$ kubectl label nodes node-1 size=Large
```

- Sau khi đã gán nhãn cho **Node** và định nghĩa **Node Selector** trong **Pod**, bạn có thể tạo **Pod** bằng cách sử dụng:

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

```bash
$ kubectl create -f pod-definition.yml
```

#### Node Selector - Hạn chế

Mặc dù **Node Selector** rất hữu ích, nhưng nó chỉ hỗ trợ lọc dựa trên các nhãn đơn giản. Đối với các yêu cầu phức tạp hơn, Kubernetes cung cấp các cơ chế **Node Affinity** và **Anti Affinity**, cho phép bạn tạo ra các quy tắc lên lịch phức tạp hơn, bao gồm cả các điều kiện "nếu thì" và ưu tiên.

#### Tài liệu tham khảo K8s:

- [Node Selectors trong Kubernetes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector)
