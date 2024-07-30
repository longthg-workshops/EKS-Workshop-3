---
title: "Fargate"
date: "`r Sys.Date()`"
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

- Tạo một vai trò IAM để sử dụng bởi Fargate

Trong module trước đó, chúng ta đã thấy cách cung cấp các máy tính EC2 để chạy Pods trong cụm EKS của chúng ta, và cách các nhóm nút được quản lý giúp giảm bớt gánh nặng vận hành. Tuy nhiên, trong mô hình này, bạn vẫn phải chịu trách nhiệm về tính sẵn có, công suất và bảo dưỡng của cơ sở hạ tầng cơ bản.

[AWS Fargate](https://aws.amazon.com/fargate/) là một công nghệ cung cấp dung lượng tính toán được kích thước đúng theo nhu cầu cho các container. Với AWS Fargate, bạn không cần phải cấu hình, cấu hình hoặc mở rộng các nhóm máy ảo một cách tự động để chạy các container. Bạn cũng không cần phải chọn loại máy chủ, quyết định khi nào để mở rộng các nhóm nút của bạn, hoặc tối ưu hóa việc đóng gói cụm. Bạn có thể kiểm soát các Pod nào bắt đầu trên Fargate và cách chúng chạy với các hồ sơ Fargate. Các hồ sơ Fargate được xác định như một phần của cụm Amazon EKS của bạn.

![EKS](/images/4/00012.png?featherlight=false&width=90pc)

Amazon EKS tích hợp Kubernetes với AWS Fargate bằng cách sử dụng các bộ điều khiển được xây dựng bởi AWS bằng cách sử dụng mô hình mở rộng được cung cấp bởi Kubernetes. Những bộ điều khiển này chạy như một phần của mặt trận điều khiển Kubernetes quản lý của Amazon EKS và chịu trách nhiệm về lập lịch các Pod Kubernetes nguyên bản vào Fargate. Các bộ điều khiển Fargate bao gồm một trình lập lịch mới chạy song song với trình lập lịch Kubernetes mặc định cùng với một số bộ điều khiển nhập và xác nhận. Khi bạn bắt đầu một Pod đáp ứng các tiêu chí để chạy trên Fargate, các bộ điều khiển Fargate đang chạy trong cụm nhận ra, cập nhật và lên lịch cho Pod trên Fargate.

Các lợi ích của Fargate bao gồm:

- AWS Fargate cho phép bạn tập trung vào ứng dụng của bạn. Bạn xác định nội dung ứng dụng, mạng, lưu trữ và yêu cầu co dãn của bạn. Không cần phải cấu hình, vá lỗi, quản lý công suất cụm, hoặc quản lý cơ sở hạ tầng.
- AWS Fargate hỗ trợ tất cả các trường hợp sử dụng container phổ biến bao gồm ứng dụng kiến trúc microservices, xử lý theo lô, ứng dụng học máy và di chuyển ứng dụng từ trên cơ sở sang đám mây.
- Chọn AWS Fargate vì mô hình cô lập và bảo mật của nó. Bạn cũng nên chọn Fargate nếu bạn muốn khởi chạy các container mà không cần phải cấu hình hoặc quản lý các máy EC2. Nếu bạn cần kiểm soát lớn hơn đối với các máy EC2 của bạn hoặc các tùy chọn tùy chỉnh rộng hơn, sau đó sử dụng ECS hoặc EKS mà không có Fargate.
