---
title: "Chạy một workloads trên Spot"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 5.3 </b>"
---

#### Chạy một workloads trên Spot

Tiếp theo, chúng ta sẽ sửa đổi ứng dụng cửa hàng bán lẻ mẫu của chúng ta để chạy thành phần catalog trên các Spot instances mới được tạo ra. Để làm điều này, chúng ta sẽ sử dụng Kustomize để áp dụng một bản vá vào `catalog` Deployment, thêm một trường `nodeSelector` với `eks.amazonaws.com/capacityType: SPOT`.

```kustomization
modules/fundamentals/mng/spot/deployment/deployment.yaml
Deployment/catalog
```

Áp dụng patch Kustomize với lệnh sau.

```bash
$ kubectl apply -k ~/environment/eks-workshop/modules/fundamentals/mng/spot/deployment

namespace/catalog không thay đổi
serviceaccount/catalog không thay đổi
configmap/catalog không thay đổi
secret/catalog-db không thay đổi
service/catalog không thay đổi
service/catalog-mysql không thay đổi
deployment.apps/catalog được cấu hình
statefulset.apps/catalog-mysql không thay đổi
```

Đảm bảo việc triển khai ứng dụng của bạn thành công với lệnh sau.

```bash
$ kubectl rollout status deployment/catalog -n catalog --timeout=5m
```

Cuối cùng, hãy xác minh rằng các pod catalog đang chạy trên các Spot instances. Chạy hai lệnh sau.

```bash
$ kubectl get pods -l app.kubernetes.io/component=service -n catalog -o wide

NAME                       READY   STATUS    RESTARTS   AGE     IP              NODE
catalog-6bf46b9654-9klmd   1/1     Running   0          7m13s   10.42.118.208   ip-10-42-99-254.us-east-2.compute.internal
$ kubectl get nodes -l eks.amazonaws.com/capacityType=SPOT

NAME                                          STATUS   ROLES    AGE   VERSION
ip-10-42-139-140.us-east-2.compute.internal   Ready    <none>   16m   vVAR::KUBERNETES_NODE_VERSION
ip-10-42-99-254.us-east-2.compute.internal    Ready    <none>   16m   vVAR::KUBERNETES_NODE_VERSION

```

Lệnh đầu tiên cho chúng ta biết rằng pod catalog đang chạy trên node `ip-10-42-99-254.us-east-2.compute.internal`, mà chúng ta xác minh là một Spot instance bằng cách so khớp nó với đầu ra của lệnh thứ hai.

Trong lab này, bạn triển khai một nhóm node quản lý tạo ra các Spot instances, và sau đó sửa đổi deployment `catalog` để chạy trên các Spot instances mới tạo ra. Theo quy trình này, bạn có thể sửa đổi bất kỳ deployment nào đang chạy trên cụm bằng cách thêm tham số `nodeSelector`, như đã chỉ định trong patch Kustomization ở trên.