---
title: "Spot instances"

weight: 5
chapter: false
pre: "<b> 5. </b>"
---


#### Spot instances


Chuẩn bị môi trường cho phần này

```bash
$ prepare-environment fundamentals/mng/spot
```

Tất cả các **node** tính toán hiện tại của chúng tôi đều đang sử dụng Dung lượng **On-Demand**. Tuy nhiên, có nhiều "[tùy chọn mua](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-purchasing-options.html)" khả dụng cho khách hàng EC2 để chạy các EKS workloads của họ.

Trong bài thực hành này, chúng ta sẽ xem xét cách chúng ta có thể tận dụng dung lượng EC2 **Spot** với các Amazon EKS Managed Node Groups.