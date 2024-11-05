---
title: "Chuẩn Bị môi trường cho lab này"

weight: 4
chapter: false
pre: "<b> 2.4 </b>"
---

Thực hiện lệnh sau trên IDE đã tạo trong bài đầu tiên:

{{% notice note %}}
```bash timeout=600 wait=30
$ prepare-environment fundamentals/mng/basics
```
{{% /notice %}}

Trong lab "Bắt đầu", chúng ta đã triển khai ứng dụng mẫu của mình lên **EKS** và đã thấy các **Pod** đang chạy. Nhưng các **Pod** này đang chạy ở đâu?

Chúng ta có thể kiểm tra managed node group mặc định đã được chuẩn bị trước cho bạn:

```bash
$ eksctl get nodegroup --cluster $EKS_CLUSTER_NAME --name $EKS_DEFAULT_MNG_NAME
```

Có một số thuộc tính của các managed node group mà chúng ta có thể thấy từ đầu ra này:

- Cấu hình về số lượng tối thiểu, tối đa và mong muốn của số lượng **node** trong nhóm này
- Loại instance cho managed node group này là `m5.large`
- Sử dụng loại **EKS AMI** `AL2_x86_64`

Chúng ta cũng có thể kiểm tra các **node** và vị trí trong các AZ.

```bash
$ kubectl get nodes -o wide --label-columns topology.kubernetes.io/zone
```

Bạn nên thấy:

- Các **node** được phân phối trên nhiều subnet trong các AZ khác nhau, cung cấp tính sẵn sàng cao

Trong suốt module này, chúng ta sẽ thực hiện các thay đổi lên managed node group này để cho thấy các khả năng cơ bản của các **MNGs**.
