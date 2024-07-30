---
title: "Tạo Graviton Nodes"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 4.1 </b>"
---

#### Tạo Graviton Nodes

Trong bài tập này, chúng ta sẽ tạo một Amazon EKS Managed Node Groups riêng biệt với các instance dựa trên Graviton và áp dụng một taint cho nó.

Đầu tiên, hãy xác nhận trạng thái hiện tại của các node có sẵn trong cụm của chúng ta:

```bash
$ kubectl get nodes -L kubernetes.io/arch
NAME                                           STATUS   ROLES    AGE     VERSION                ARCH
ip-192-168-102-2.us-west-2.compute.internal    Ready    <none>   6h56m   vVAR::KUBERNETES_NODE_VERSION    amd64
ip-192-168-137-20.us-west-2.compute.internal   Ready    <none>   6h56m   vVAR::KUBERNETES_NODE_VERSION    amd64
ip-192-168-19-31.us-west-2.compute.internal    Ready    <none>   6h56m   vVAR::KUBERNETES_NODE_VERSION    amd64
```

Đầu ra cho thấy các node hiện có của chúng tôi với các cột hiển thị kiến trúc CPU của mỗi node. Tất cả các node này hiện đang sử dụng các node `amd64`.

Chúng ta chưa cấu hình các taints, điều này sẽ được thực hiện sau.

Lệnh sau tạo Node Groups Graviton:

```bash timeout=600 hook=configure-taints
$ aws eks create-nodegroup \
  --cluster-name $EKS_CLUSTER_NAME \
  --nodegroup-name graviton \
  --node-role $GRAVITON_NODE_ROLE \
  --subnets $PRIMARY_SUBNET_1 $PRIMARY_SUBNET_2 $PRIMARY_SUBNET_3 \
  --instance-types t4g.medium \
  --ami-type AL2_ARM_64 \
  --scaling-config minSize=1,maxSize=3,desiredSize=1 \
  --disk-size 20
```

Lệnh aws `eks wait nodegroup-active` có thể được sử dụng để chờ đợi cho đến khi một Node Groups EKS cụ thể đã hoạt động và sẵn sàng sử dụng. Lệnh này là một phần của AWS CLI và có thể được sử dụng để đảm bảo rằng Node Groups được chỉ định đã được tạo thành công và tất cả các instance liên quan đang chạy và sẵn sàng.

```bash wait=30 timeout=300
$ aws eks wait nodegroup-active \
  --cluster-name $EKS_CLUSTER_NAME \
  --nodegroup-name graviton
```


Khi Amazon EKS Managed Node Groups mới của chúng ta **Active**, chạy lệnh sau:

```bash
$ kubectl get nodes \
    --label-columns eks.amazonaws.com/nodegroup,kubernetes.io/arch

NAME                                          STATUS   ROLES    AGE    VERSION               NODEGROUP   ARCH
ip-192-168-102-2.us-west-2.compute.internal   Ready    <none>   6h56m  vVAR::KUBERNETES_NODE_VERSION   default     amd64
ip-192-168-137-20.us-west-2.compute.internal  Ready    <none>   6h56m  vVAR::KUBERNETES_NODE_VERSION   default     amd64
ip-192-168-19-31.us-west-2.compute.internal   Ready    <none>   6h56m  vVAR::KUBERNETES_NODE_VERSION   default     amd64
ip-10-42-172-231.us-west-2.compute.internal   Ready    <none>   2m5s   vVAR::KUBERNETES_NODE_VERSION   graviton    arm64
```

Lệnh trên sử dụng cờ `--selector` để truy vấn tất cả các node có labels `eks.amazonaws.com/nodegroup` khớp với tên của Amazon EKS Managed Node Groups của chúng ta `graviton`. Cờ `--label-columns` cũng cho phép chúng ta hiển thị giá trị của labels `eks.amazonaws.com/nodegroup` cũng như kiến trúc bộ xử lý trong đầu ra. Lưu ý rằng cột `ARCH` hiển thị Node Groups đã bị taint chạy Graviton `arm64`.

Hãy khám phá cấu hình hiện tại của node của chúng ta. Lệnh sau sẽ liệt kê chi tiết của tất cả các node thuộc Amazon EKS Managed Node Groups.

```bash
$ kubectl describe nodes \
    --selector eks.amazonaws.com/nodegroup=graviton
Name:               ip-10-42-12-233.us-west-2.compute.internal
Roles:              <none>
Labels:             beta.kubernetes.io/instance-type=t4g.medium
                    beta.kubernetes.io/os=linux
                    eks.amazonaws.com/capacityType=ON_DEMAND
                    eks.amazonaws.com/nodegroup=graviton
                    eks.amazonaws.com/nodegroup-image=ami-0b55230f107a87100
                    eks.amazonaws.com/sourceLaunchTemplateId=lt-07afc97c4940b6622
                    kubernetes.io/arch=arm64
                    [...]
CreationTimestamp:  Wed, 09 Nov 2022 10:36:26 +0000
Taints:             <none>
[...]
```

Một số điểm cần chú ý:

1. EKS tự động thêm một số labels nhất định để cho phép việc lọc dễ dàng hơn, bao gồm các labels cho loại hệ điều hành, tên Amazon EKS Managed Node Groups và các loại khác. Trong khi một số labels được cung cấp sẵn với EKS, AWS cho phép các nhà điều hành cấu hình bộ labels tùy chỉnh của riêng họ tại mức độ Amazon EKS Managed Node Groups. Điều này đảm bảo rằng mỗi node trong một Node Groups sẽ có các labels nhất quán. Nhãn `kubernetes.io/arch` cho thấy chúng ta đang chạy một instance EC2 với kiến trúc CPU ARM64.
2. Hiện tại không có taint được cấu hình cho node đã khám phá, như được thể hiện bởi phần `Taints: <none>`.

#### Cấu hình taint cho Amazon EKS Managed Node Groups

Trong khi dễ dàng thêm taint cho các node bằng cách sử dụng CLI `kubectl` như mô tả [tại đây](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/#concepts), một quản trị viên sẽ phải thực hiện thay đổi này mỗi khi Node Groups dưới cơ bắt đầu hoặc kết thúc. Để vượt qua thách thức này, AWS hỗ trợ thêm cả `labels` và `taints` vào các Amazon EKS Managed Node Groups, đảm bảo mọi node trong MNG sẽ có các labels và taints tương ứng được cấu hình tự động.

Bây giờ hãy thêm một taint vào Amazon EKS Managed Node Groups được thiết lập trước của chúng ta `graviton`. Taint này sẽ có `key=frontend`, `value=true` và `effect=NO_EXECUTE`. Điều này đảm bảo rằng bất kỳ pod nào đang chạy trên Amazon EKS Managed Node Groups đã bị taint của chúng tôi sẽ bị đuổi ra nếu chúng không có toleration khớp. Ngoài ra, không có pod mới nào sẽ được lên lịch trên Amazon EKS Managed Node Groups này mà không có toleration phù hợp.

Bắt đầu bằng cách thêm một `taint` vào Amazon EKS Managed Node Groups được thiết lập trước của chúng ta bằng lệnh `aws` cli sau:

```bash wait=20
$ aws eks update-nodegroup-config \
    --cluster-name $EKS_CLUSTER_NAME --nodegroup-name graviton \
    --taints "addOrUpdateTaints=[{key=frontend, value=true, effect=NO_EXECUTE}]"
{
    "update": {
        "id": "488a2b7d-9194-3032-974e-2f1056ef9a1b",
        "status": "InProgress",
        "type": "ConfigUpdate",
        "params": [
            {
                "type": "TaintsToAdd",
                "value": "[{\"effect\":\"NO_EXECUTE\",\"value\":\"true\",\"key\":\"frontend\"}]"
            }
        ],
        "createdAt": "2022-11-09T15:20:10.519000+00:00",
        "errors": []
    }
}
```

Chạy lệnh sau để chờ Node Groups trở thành hoạt động.

```bash timeout=180
$ aws eks wait nodegroup-active --cluster-name $EKS_CLUSTER_NAME \
  --nodegroup-name graviton
```

Thêm, loại bỏ hoặc thay thế taints có thể được thực hiện bằng cách sử dụng lệnh CLI [`aws eks update-nodegroup-config`](https://docs.aws.amazon.com/cli/latest/reference/eks/update-nodegroup-config.html) để cập nhật cấu hình của Amazon EKS Managed Node Groups. Điều này có thể được thực hiện bằng cách chuyển `addOrUpdateTaints` hoặc `removeTaints` và một danh sách các taints đến cờ `--taints` của lệnh.


Bạn cũng có thể cấu hình taints trên một Amazon EKS Managed Node Groups bằng cách sử dụng CLI `eksctl`. Xem [tài liệu](https://eksctl.io/usage/nodegroup-taints/) để biết thêm thông tin.

Chúng ta đã sử dụng `effect=NO_EXECUTE` trong cấu hình taint của chúng ta. Hiện tại, các Amazon EKS Managed Node Groups hỗ trợ các giá trị sau cho taint `effect`:

- `NO_SCHEDULE` - Tương ứng với hiệu ứng taint Kubernetes `NoSchedule`. Điều này cấu hình Amazon EKS Managed Node Groups với một taint đẩy lùi tất cả các pod không có toleration khớp. Tất cả các pod đang chạy không bị đuổi ra khỏi các node của Amazon EKS Managed Node Groups.
- `NO_EXECUTE` - Tương ứng với hiệu ứng taint Kubernetes `NoExecute`. Cho phép các node được cấu hình với taint này không chỉ đẩy lùi các pod mới được lên lịch mà còn đuổi ra bất kỳ pod nào đang chạy mà không có toleration khớp.
- `PREFER_NO_SCHEDULE` - Tương ứng với hiệu ứng taint Kubernetes `PreferNoSchedule`. Nếu có thể, EKS tránh lên lịch các Pod không chịu taint này vào node.

Chúng ta có thể sử dụng lệnh sau để kiểm tra taint đã được cấu hình đúng cho Amazon EKS Managed Node Groups:

```bash
$ aws eks describe-nodegroup --cluster-name $EKS_CLUSTER_NAME \
  --nodegroup-name graviton \
  | jq .nodegroup.taints
[
  {
    "key": "frontend",
    "value": "true",
    "effect": "NO_EXECUTE"
  }
]
```



Cập nhật Amazon EKS Managed Node Groups và lan truyền các labels và taints thường mất vài phút. Nếu bạn không thấy bất kỳ taint nào được cấu hình hoặc nhận được giá trị `null`, hãy đợi một vài phút trước khi thử lại lệnh trên.


Xác minh bằng lệnh CLI `kubectl`, chúng ta cũng có thể thấy rằng taint đã được lan truyền đúng cho node tương ứng:

```bash
$ kubectl describe nodes \
   

 --selector eks.amazonaws.com/nodegroup=graviton | grep Taints
Taints:             frontend=true:NoExecute
```