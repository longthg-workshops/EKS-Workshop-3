---
title: "Thêm các node"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 3.1 </b>"
---

#### Cập nhật cấu hình Amazon EKS Managed Node Groups trong Amazon EKS

Khi làm việc với cluster của bạn, có thể bạn cần cập nhật cấu hình Amazon EKS Managed Node Groups để thêm các node bổ sung để hỗ trợ nhu cầu của công việc của bạn. Có nhiều cách để mở rộng một node group, trong trường hợp của chúng ta, chúng ta sẽ sử dụng lệnh `eksctl scale nodegroup`.

Đầu tiên, hãy truy xuất cấu hình mở rộng node group hiện tại và xem **kích thước tối thiểu**, **kích thước tối đa** và **sức chứa mong muốn** của các node bằng cách sử dụng lệnh `eksctl` dưới đây:

```bash
$ eksctl get nodegroup --name $EKS_DEFAULT_MNG_NAME --cluster $EKS_CLUSTER_NAME
```

Chúng ta sẽ mở rộng node group trong `eks-workshop` bằng cách thay đổi số lượng node từ `3` thành `4` cho **sức chứa mong muốn** bằng lệnh dưới đây:

```bash
$ eksctl scale nodegroup --name $EKS_DEFAULT_MNG_NAME --cluster $EKS_CLUSTER_NAME \
    --nodes 4 --nodes-min 3 --nodes-max 6
```

Một node group cũng có thể được cập nhật bằng cách sử dụng `aws` CLI với lệnh sau đây. Xem [tài liệu](https://docs.aws.amazon.com/cli/latest/reference/eks/update-nodegroup-config.html) để biết thêm thông tin.

```
aws eks update-nodegroup-config --cluster-name $EKS_CLUSTER_NAME \
  --nodegroup-name $EKS_DEFAULT_MNG_NAME --scaling-config minSize=4,maxSize=6,desiredSize=4
```

Sau khi thay đổi node group, có thể mất tới **2-3 phút** để các thay đổi cấu hình cung cấp và cấu hình node có hiệu lực. Hãy truy xuất cấu hình node group một lần nữa và xem **kích thước tối thiểu**, **kích thước tối đa** và **sức chứa mong muốn** của các node bằng cách sử dụng lệnh `eksctl` dưới đây:

```bash
$ eksctl get nodegroup --name $EKS_DEFAULT_MNG_NAME --cluster $EKS_CLUSTER_NAME
```

Để đợi cho đến khi hoạt động cập nhật node group hoàn tất, bạn có thể chạy lệnh này:

```bash hook=wait-node
$ aws eks wait nodegroup-active --cluster-name $EKS_CLUSTER_NAME --nodegroup-name $EKS_DEFAULT_MNG_NAME
```

Khi lệnh trên hoàn tất, chúng ta có thể xem lại số lượng node worker đã thay đổi bằng lệnh sau, liệt kê tất cả các node trong Amazon EKS Managed Node Groups của chúng ta bằng cách sử dụng nhãn như một bộ lọc:

Có thể mất khoảng một phút để node xuất hiện trong đầu ra dưới đây, nếu danh sách vẫn hiển thị 3 node, hãy kiên nhẫn.


```bash
$ kubectl get nodes -l eks.amazonaws.com/nodegroup=$EKS_DEFAULT_MNG_NAME
NAME                                          STATUS     ROLES    AGE     VERSION
ip-10-42-104-151.us-west-2.compute.internal   Ready      <none>   2d23h   vVAR::KUBERNETES_NODE_VERSION
ip-10-42-144-11.us-west-2.compute.internal    Ready      <none>   2d23h   vVAR::KUBERNETES_NODE_VERSION
ip-10-42-146-166.us-west-2.compute.internal   NotReady   <none>   18s     vVAR::KUBERNETES_NODE_VERSION
ip-10-42-182-134.us-west-2.compute.internal   Ready      <none>   2d23h   vVAR::KUBERNETES_NODE_VERSION
```

Lưu ý rằng node hiển thị trạng thái `NotReady`, điều này xảy ra khi node mới vẫn đang trong quá trình tham gia vào cluster. Chúng ta cũng có thể sử dụng `kubectl wait` để chờ đến khi tất cả các node báo cáo `Ready`:

```bash hook=add-node
$ kubectl wait --for=condition=Ready nodes --all --timeout=300s
```
