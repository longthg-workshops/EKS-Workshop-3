---
title: "Bật Fargate"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 6.1 </b>"
---

#### Cấu hình Fargate Profile cho EKS

Trước khi bạn lên lịch chạy các Pods trên Fargate trong cluster của mình, bạn cần định nghĩa ít nhất một Fargate profile để xác định các Pods sử dụng Fargate khi được khởi chạy.

Là một quản trị viên, bạn có thể sử dụng Fargate profile để khai báo các Pods nào chạy trên Fargate. Bạn có thể thực hiện điều này thông qua các bộ chọn (selectors) trong profile. Bạn có thể thêm tối đa năm bộ chọn cho mỗi profile. Mỗi bộ chọn bao gồm một namespace và các nhãn (labels) tùy chọn. Bạn cần phải định nghĩa một namespace cho mỗi bộ chọn. Trường nhãn bao gồm nhiều cặp khóa-giá trị tùy chọn. Các Pods khớp với một bộ chọn sẽ được lên lịch chạy trên Fargate. Pods được khớp sử dụng namespace và các nhãn được chỉ định trong bộ chọn. Nếu một bộ chọn namespace được định nghĩa mà không có nhãn, Amazon EKS sẽ cố gắng lên lịch chạy tất cả các Pods trong namespace đó lên Fargate sử dụng profile. Nếu một Pod được lên lịch chạy khớp với bất kỳ bộ chọn nào trong Fargate profile, thì Pod đó sẽ được lên lịch chạy trên Fargate.

Nếu một Pod khớp với nhiều Fargate profiles, bạn có thể chỉ định profile nào mà Pod sử dụng bằng cách thêm nhãn Kubernetes sau đây vào đặc tả Pod: `eks.amazonaws.com/fargate-profile: my-fargate-profile`. Pod phải khớp với một bộ chọn trong profile đó để được lên lịch chạy trên Fargate. Các quy tắc ưu tiên/anti-ưu tiên của Kubernetes không áp dụng và không cần thiết với các Pod Fargate của Amazon EKS.

Bây giờ, hãy bắt đầu bằng cách thêm một Fargate profile vào cluster EKS của chúng ta. Đây là cấu hình `eksctl` mà chúng ta sẽ sử dụng:

```file
manifests/modules/fundamentals/fargate/profile/fargate.yaml
```

Cấu hình này tạo ra một Fargate profile có tên là `checkout-profile` với các đặc điểm sau:

1. Mục tiêu là các Pods trong namespace `checkout` có nhãn `fargate: yes`
2. Đặt pod trong các private subnet của VPC
3. Áp dụng một IAM role cho cơ sở hạ tầng Fargate để nó có thể kéo các images từ ECR, ghi log vào CloudWatch, và như thế.

Lệnh sau tạo profile, mất vài phút:

```bash
$ cat ~/environment/eks-workshop/modules/fundamentals/fargate/profile/fargate.yaml \
| envsubst \
| eksctl create fargateprofile -f -
```

Bây giờ chúng ta có thể kiểm tra Fargate profile:

```bash
$ aws eks describe-fargate-profile \
    --cluster-name $EKS_CLUSTER_NAME \
    --fargate-profile-name checkout-profile
{
    "fargateProfile": {
        "fargateProfileName": "checkout-profile",
        "fargateProfileArn": "arn:aws:eks:us-west-2:1234567890:fargateprofile/eks-workshop/checkout-profile/92c4e2e3-50cd-773c-1c32-52e4d44

cd0ca",
        "clusterName": "eks-workshop",
        "createdAt": "2023-08-05T12:57:58.022000+00:00",
        "podExecutionRoleArn": "arn:aws:iam::1234567890:role/eks-workshop-fargate",
        "subnets": [
            "subnet-01c3614cdd385a93c",
            "subnet-0e392224ce426565a",
            "subnet-07f8a6fda62ec83df"
        ],
        "selectors": [
            {
                "namespace": "checkout",
                "labels": {
                    "fargate": "yes"
                }
            }
        ],
        "status": "ACTIVE",
        "tags": {}
    }
}
