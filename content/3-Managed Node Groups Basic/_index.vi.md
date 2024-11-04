---
title: "Amazon EKS Managed Node Groups"

weight: 3
chapter: false
pre: "<b> 3. </b>"
---

{{% notice note %}}
Chuẩn bị môi trường cho phần này:
```bash
$ prepare-environment fundamentals/mng/basics
```
{{% /notice %}}

Trong các bài thực hành khác của EKS, chúng ta đã triển khai ứng dụng mẫu của mình cho EKS và thấy các Pod đang chạy. Nhưng các Pod này đang chạy ở đâu?

Ta có thể kiểm tra _Managed node group_ mặc định được cung cấp trước cho bạn:
```bash
$ eksctl get nodegroup --cluster $EKS_CLUSTER_NAME --name $EKS_DEFAULT_MNG_NAME
```

Có một số thuộc tính của nhóm các node được quản lý mà chúng ta có thể thấy từ đầu ra này:

- Cấu hình số lượng tối thiểu, tối đa và mong muốn của số lượng nút trong nhóm này
- Kiểu thể hiện cho nhóm các node này là `m5.large`
- Sử dụng kiểu AMI EKS `AL2_x86_64`

Chúng ta cũng có thể kiểm tra các nút và vị trí trong vùng khả dụng (Availability Zone - AZ).

```bash
$ kubectl get nodes -o wide --label-columns topology.kubernetes.io/zone
```

Bạn sẽ thấy:

- Các nút được phân bổ trên nhiều mạng con trong nhiều AZ khác nhau, mang lại tính khả dụng cao

Trong suốt mô-đun này, chúng tôi sẽ thực hiện các thay đổi đối với nhóm các node này để chứng minh các khả năng cơ bản của MNG.