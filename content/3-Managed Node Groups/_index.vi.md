---
title: "Amazon EKS Managed Node Groups"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 3. </b>"
---

#### Tạo Cluster EKS và Quản lý Node Sử dụng Amazon EKS Managed Node Groups

Một **cluster EKS** chứa một hoặc nhiều **node EC2** mà các **Pod** được lên lịch trên đó. Các **node EKS** chạy trong tài khoản AWS của bạn và kết nối với mặt điều khiển của cụm của bạn thông qua điểm cuối máy chủ API của cụm. Bạn triển khai một hoặc nhiều node vào một **node group**. Một node group là một hoặc nhiều trường hợp EC2 được triển khai trong một EC2 AutoScaling.

Các **node EKS** là các trường hợp tiêu chuẩn của Amazon EC2. Bạn được tính phí cho chúng dựa trên giá của EC2. Để biết thêm thông tin, hãy xem [Giá của Amazon EC2](https://aws.amazon.com/ec2/pricing/).

[Các node group được quản lý của Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html) tự động hóa việc cung cấp và quản lý vòng đời của các node cho các cụm Amazon EKS. Điều này rất giúp đỡ trong các hoạt động vận hành như cập nhật liên tục cho các AMI mới hoặc triển khai phiên bản Kubernetes mới.

![EKS](/images/4/00013.png?featherlight=false&width=30pc)

Các ưu điểm của việc chạy các node group được quản lý của Amazon EKS bao gồm:

- Tạo, cập nhật tự động hoặc chấm dứt các node với một hoạt động duy nhất bằng cách sử dụng bảng điều khiển Amazon EKS, `eksctl`, AWS CLI, AWS API, hoặc các công cụ mã hạ tầng như AWS CloudFormation và Terraform.
- Các node được cung cấp chạy bằng các AMI tối ưu mới nhất của Amazon EKS.
- Các node được cung cấp như là một phần của MNG tự động được gắn thẻ với siêu dữ liệu như AZ, kiến trúc CPU và loại trường hợp.
- Cập nhật và chấm dứt node tự động và một cách an toàn giúp đảm bảo rằng ứng dụng của bạn vẫn khả dụng.
- Không có chi phí bổ sung để sử dụng các node group được quản lý của Amazon EKS, chỉ thanh toán cho các tài nguyên AWS được cung cấp.

Các thực hành trong phần này đề cập đến các cách khác nhau mà các node group được quản lý của EKS có thể được sử dụng để cung cấp khả năng tính toán cho một cụm.

