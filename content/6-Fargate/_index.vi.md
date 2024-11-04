---
title: "Fargate"

weight: 6
chapter: false
pre: "<b> 6. </b>"
---

#### Fargate

#### Chuẩn bị môi trường cho phần này:

```bash timeout=400 wait=30
$ prepare-environment fundamentals/fargate
```

Điều này sẽ thực hiện các thay đổi sau vào môi trường thực hành của bạn:

- Tạo một IAM Role để sử dụng bởi Fargate

Trong module trước đó, chúng ta đã thấy cách cung cấp các máy tính EC2 để chạy Pods trong cụm EKS của chúng ta, và cách các nhóm nút được quản lý giúp giảm bớt gánh nặng vận hành. Tuy nhiên, mô hình trên vẫn yêu cầu bạn phải chịu trách nhiệm về tính sẵn có, công suất và bảo dưỡng của cơ sở hạ tầng cơ bản.

Trong phần này, chúng ta sẽ tìm hiểu cách bật Fargate, sau đó thực hiện các công việc xếp lịch, cấp phát tài nguyên và mở rộng khối công việc chạy trên Fargate. Fargate là công nghệ tính toán phi máy chủ, giúp bạn giảm sự quan tâm đến việc cấp phát tài nguyên phần cứng cho hệ thống.