---
title: "Amazon Elastic Kubernetes Service - Tính toán"
weight: 1
chapter: false
---

# Amazon Elastic Kubernetes Service - Tính toán

![EKS Arch](../images/home/Architecture-of-EKS.webp)

### Tổng quan

Một **cụm EKS _(EKS Cluster)_** chứa một hoặc nhiều **node EC2**, nơi các **Pod** được lên lịch và vẫn hành. Các **node EKS** chạy trong tài khoản AWS của bạn và kết nối với tầng điều khiển của cụm của bạn thông qua endpoint của API-Server.

Các **node EKS** có thể là:

- **Các máy ảo Amazon EC2:** Chúng được tính phí theo giá tiêu chuẩn của EC2 (Xem [Phí dịch vụ Amazon EC2](https://aws.amazon.com/ec2/pricing/)). Ta có thể triển khai một hoặc nhiều node vào một **node group**. Một node group là một hoặc nhiều máy ảo EC2 được triển khai trong một **EC2 AutoScaling Group**.

- **AWS Fargate:** Dịch vụ tính toán phi máy chủ, cung cấp khả năng tính toán theo yêu cầu, đúng kích cỡ cho các container. Với AWS Fargate, bạn không phải tự cung cấp, tự cấu hình hay mở rộng tài nguyên EC2 để chạy container. Bạn cũng không cần phải chọn loại server hay quyết định cách thức AutoScaling. Thay vào đó, bạn có thể kiểm soát việc khởi tạo các Pod trên Fargate và cách chúng chạy với các cấu hình Fargate.

### Mục tiêu bài lab
Trong bài lab này, chúng ta sẽ tìm hiểu về một số khái niệm và tài nguyên tính toán trên Amazon EKS, bao gồm:

- Managed Node Group
- Quản lý và nâng cấp AMI
- Graviton ARM Instance
- Tiết kiệm chi phí với các Spot Instance
- Triển khai các pod trên AWS Fargate