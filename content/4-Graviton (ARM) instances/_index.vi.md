---
title: "Graviton (ARM) instances"

weight: 4
chapter: false
pre: "<b> 4. </b>"
---

### Graviton (ARM) instances

Chuẩn bị môi trường cho phần này:

```bash timeout=600 wait=30
$ prepare-environment fundamentals/mng/graviton
```

Cho dù bạn đang sử dụng các phiên bản **On-demand** hoặc **Spot instances**, **AWS** cung cấp 3 loại bộ xử lý cho **EC2** cũng như các Amazon EKS Managed Node Groups **EC2-backed EKS**. Khách hàng có sự lựa chọn của bộ xử lý **Intel**, **AMD** và **ARM (AWS Graviton)**. [Bộ xử lý AWS Graviton](https://aws.amazon.com/ec2/graviton/) được thiết kế bởi **AWS** để cung cấp hiệu suất giá tốt nhất cho các khối công việc đám mây của bạn chạy trên **Amazon EC2**.

Các instance dựa trên **Graviton** có thể được xác định bằng chữ `g` trong phần **Processor family** của [Qui tắc đặt tên loại instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html#instance-type-names).

![EKS](/images/4/00010.png?featherlight=false&width=30pc)

### Taints và Toleration với các node Graviton

**Taints** là một thuộc tính của một node để đẩy lùi một số pods nhất định. **Tolerations** được áp dụng cho các pods để cho phép chúng được lên lịch trên các node với **taints** phù hợp. **Taints** và **tolerations** hoạt động cùng nhau để đảm bảo rằng các pods không được lên lịch trên các node không phù hợp.

Cấu hình các node **tainted** hữu ích trong các tình huống khi chúng ta cần đảm bảo rằng chỉ các pods cụ thể được lên lịch trên các nhóm node cụ thể có phần cứng đặc biệt (như các instance dựa trên **Graviton** hoặc **GPU** đính kèm). Trong bài tập thực hành này, chúng ta sẽ tìm hiểu cách cấu hình **taints** cho các Amazon EKS Managed Node Groups của chúng ta và cách thiết lập ứng dụng của chúng ta để sử dụng các node **tainted** chạy các bộ xử lý dựa trên **Graviton**.
