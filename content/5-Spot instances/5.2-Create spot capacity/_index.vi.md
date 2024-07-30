---
title: "Tạo dung lượng dạng Spot"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 5.2 </b>"
---

#### Tạo dung lượng dạng Spot

Chúng ta sẽ triển khai một Amazon EKS Managed Node Groups tạo ra các instance Spot, sau đó sửa đổi thành phần `catalog` hiện có của ứng dụng của chúng ta để chạy trên các instance Spot mới tạo.

Chúng ta có thể bắt đầu bằng cách liệt kê tất cả các node trong cụm EKS hiện có của chúng ta. Lệnh `kubectl get nodes` có thể được sử dụng để liệt kê các node trong cụm Kubernetes của bạn, nhưng để có thêm chi tiết về loại dung lượng, chúng ta sẽ sử dụng tham số `-L eks.amazonaws.com/capacityType`.

Lệnh sau đây cho thấy rằng các node của chúng tôi hiện đang là các instance **on-demand**.

```bash
$ kubectl get nodes -L eks.amazonaws.com/capacityType
NAME                                          STATUS   ROLES    AGE    VERSION                CAPACITYTYPE
ip-10-42-103-103.us-east-2.compute.internal   Ready    <none>   133m   vVAR::KUBERNETES_NODE_VERSION    ON_DEMAND
ip-10-42-142-197.us-east-2.compute.internal   Ready    <none>   133m   vVAR::KUBERNETES_NODE_VERSION    ON_DEMAND
ip-10-42-161-44.us-east-2.compute.internal    Ready    <none>   133m   vVAR::KUBERNETES_NODE_VERSION    ON_DEMAND
```

Nếu bạn muốn lấy các node dựa trên một loại dung lượng cụ thể, như các instance `on-demand`, bạn có thể sử dụng "<b>label selectors</b>". Trong tình huống cụ thể này, bạn có thể đạt được điều này bằng cách đặt label selector thành `capacityType=ON_DEMAND`.

```bash
$ kubectl get nodes -l eks.amazonaws.com/capacityType=ON_DEMAND

NAME                                         STATUS   ROLES    AGE     VERSION
ip-10-42-10-119.us-east-2.compute.internal   Ready    <none>   3d10h   vVAR::KUBERNETES_NODE_VERSION
ip-10-42-10-200.us-east-2.compute.internal   Ready    <none>   3d10h   vVAR::KUBERNETES_NODE_VERSION
ip-10-42-11-94.us-east-2.compute.internal    Ready    <none>   3d10h   vVAR::KUBERNETES_NODE_VERSION
ip-10-42-12-235.us-east-2.compute.internal   Ready    <none>   4h34m   vVAR::KUBERNETES_NODE_VERSION
```


Trong sơ đồ dưới đây, có hai "Node Groups" riêng biệt đại diện cho các Amazon EKS Managed Node Groups trong cụm. Hộp Node Groups đầu tiên đại diện cho Node Groups chứa các instance On-Demand trong khi hộp thứ hai đại diện cho Node Groups chứa các instance Spot. Cả hai đều liên kết với cụm EKS được chỉ định.

![spot arch](./assets/managed-spot-arch.png)

Hãy tạo một Node Groups với các instance Spot. Lệnh sau tạo ra một Node Groups mới có tên `managed-spot`.

```bash wait=10
$ aws eks create-nodegroup \
  --cluster-name $EKS_CLUSTER_NAME \
  --nodegroup-name managed-spot \
  --node-role $SPOT_NODE_ROLE \
  --subnets $PRIMARY_SUBNET_1 $PRIMARY_SUBNET_2 $PRIMARY_SUBNET_3 \
  --instance-types c5.large c5d.large c5a.large c5ad.large c6a.large \
  --capacity-type SPOT \
  --scaling-config minSize=2,maxSize=3,desiredSize=2 \
  --disk-size 20
```

Đối số `--capacity-type SPOT` chỉ định rằng tất cả dung lượng trong Amazon EKS Managed Node Groups này nên là Spot.


Lệnh aws `eks wait nodegroup-active` có thể được sử dụng để đợi cho đến khi một Node Groups EKS cụ thể đã hoạt động và sẵn sàng sử dụng. Lệnh này là một phần của AWS CLI và có thể được sử dụng để đảm bảo rằng Node Groups được chỉ định đã được tạo thành công và tất cả các instance liên quan đều đang chạy và sẵn sàng.

```bash wait=30 timeout=300
$ aws eks wait nodegroup-active \
  --cluster-name $EKS_CLUSTER_NAME \
  --nodegroup-name managed-spot
```


Khi Amazon EKS Managed Node Groups mới của chúng tôi là **Active**, chạy lệnh sau.

```bash
$ kubectl get nodes -L eks.amazonaws.com/capacityType,eks.amazonaws.com/nodegroup

NAME                                          STATUS   ROLES    AGE     VERSION                CAPACITYTYPE   NODEGROUP
ip-10-42-103-103.us-east-2.compute.internal   Ready    <none>   3h38m   vVAR::KUBERNETES_NODE_VERSION    ON_DEMAND      default
ip-10-42-142-197.us-east-2.compute.internal   Ready    <none>   3h38m   vVAR::KUBERNETES_NODE_VERSION    ON_DEMAND      default
ip-10-42-161-44.us-east-2.compute.internal    Ready    <none>   3h38m   vVAR::KUBERNETES_NODE_VERSION    ON_DEMAND      default
ip-10-42-178-46.us-east-2.compute.internal    Ready    <none>   103s    vVAR::KUBERNETES_NODE_VERSION    SPOT           managed-spot
ip-10-42-97-19.us-east-2.compute.internal     Ready    <none>   104s    vVAR::KUBERNETES_NODE_VERSION    SPOT           managed-spot
```

Đầu ra cho thấy rằng hai node bổ sung đã được cung cấp trong Node Groups `managed-spot` với loại dung lượng là `SPOT`.