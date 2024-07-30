---
title: "Spot instances"
date: "`r Sys.Date()`"
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

Một **Spot Instance** sử dụng dung lượng EC2 dự phòng có sẵn với giá thấp hơn giá **On-Demand**. Bởi vì **Spot Instances** cho phép bạn yêu cầu các máy EC2 không sử dụng với giảm giá đáng kể, bạn có thể giảm chi phí Amazon EC2 của mình đáng kể. Giá hàng giờ cho một **Spot Instance** được gọi là giá **Spot**. Giá **Spot** của mỗi loại **instance** trong mỗi AZ được đặt bởi Amazon EC2, và được điều chỉnh dần dần dựa trên cung cấp và cầu cảng dài hạn cho các **Spot Instances**. **Spot Instance** của bạn chạy bất cứ khi nào có dung lượng có sẵn.

**Spot Instances** là lựa chọn phù hợp cho các ứng dụng linh hoạt, không trạng thái, có khả năng chịu lỗi. Điều này bao gồm các công việc đào tạo hệ thống phân tán và học máy, các công việc **ETL** dữ liệu lớn như Apache Spark, các ứng dụng xử lý hàng đợi và các **API** endpoint không trạng thái. Bởi vì **Spot** là dung lượng EC2 dự phòng, có thể thay đổi theo thời gian, chúng tôi khuyến nghị bạn sử dụng dung lượng **Spot** cho các công việc có khả năng chịu gián đoạn. Cụ thể hơn, dung lượng **Spot** phù hợp cho các công việc có thể chịu được các thời gian mà dung lượng cần không có sẵn.

Trong bài thực hành này, chúng ta sẽ xem xét cách chúng ta có thể tận dụng dung lượng EC2 **Spot** với các Amazon EKS Managed Node Groups.