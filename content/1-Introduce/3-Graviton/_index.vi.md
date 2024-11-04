---
title: "AWS Graviton Instance"

weight: 2
chapter: false
pre: "<b> 1.2 </b>"
---

Các bộ xử lý **AWS Graviton** được xây dựng trên [Hệ thống AWS Nitro](https://aws.amazon.com/ec2/nitro/?p=pm&pd=graviton&z=3). **AWS** đã xây dựng **Hệ thống AWS Nitro** để gần như tất cả các tài nguyên tính toán và bộ nhớ của phần cứng máy chủ được giao cho các instance của bạn. Điều này được đạt được bằng cách phân rã các chức năng **hypervisor** và khả năng quản lý từ các máy chủ và giao chúng cho phần cứng và phần mềm có chức năng riêng biệt. Điều này dẫn đến hiệu suất tổng thể tốt hơn, khác với các nền tảng ảo hóa truyền thống chạy phần mềm **hypervisor** trên cùng một máy chủ vật lý như các máy ảo, điều này có nghĩa là các máy ảo không thể sử dụng 100% tài nguyên của máy chủ. **Hệ thống AWS Nitro** được hỗ trợ bởi các hệ điều hành **Linux** phổ biến cùng với nhiều ứng dụng và dịch vụ phổ biến từ **AWS** và các **Nhà cung cấp Phần mềm Độc lập**.

### Mô hình đa kiến trúc với Bộ xử lý Graviton

**AWS Graviton** yêu cầu container images tương thích với **ARM**, lý tưởng là đa kiến trúc (**ARM64** và **AMD64**) cho phép tương thích chéo với cả các loại instance **Graviton** và **x86**.

Các bộ xử lý **Graviton** tăng cường trải nghiệm **EKS** cho các Amazon EKS Managed Node Groups với các instance mang lại giá thấp hơn đến 20%, hiệu suất giá tốt hơn đến 40%, và tiêu thụ năng lượng ít hơn đến 60% so với các instance **x86** thế hệ thứ năm tương đương. Các Amazon EKS Managed Node Groups **EKS** dựa trên **Graviton** khởi động một nhóm **EC2 Auto Scaling** với các bộ xử lý **Graviton**.

Thêm các instance dựa trên **Graviton** vào Amazon EKS Managed Node Groups **EKS** của bạn giới thiệu một cơ sở hạ tầng đa kiến trúc và nhu cầu cho ứng dụng của bạn chạy trên các CPU khác nhau. Điều này có nghĩa là mã ứng dụng của bạn cần phải có sẵn trong các triển khai **Kiến trúc Tập lệnh Hướng dẫn (ISA)** khác nhau. Có nhiều tài nguyên khác nhau để giúp các nhóm lập kế hoạch và chuyển ứng dụng sang các instance dựa trên **Graviton**. Hãy kiểm tra [kế hoạch chuyển đổi Graviton](https://pages.awscloud.com/rs/112-TZM-766/images/Graviton%20Challenge%20Plan.pdf) và [Cố vấn Chuyển đổi cho Graviton](https://github.com/aws/porting-advisor-for-graviton) để có các tài nguyên hữu ích.

![EKS](../../../images/4/00011.png?featherlight=false&width=40pc)

Kiến trúc ứng dụng web mẫu của cửa hàng bán lẻ chứa [container images đã được xây sẵn cho cả hai kiến trúc CPU **x86-64** và **ARM64**](https://gallery.ecr.aws/aws-containers/retail-store-sample-ui).

Khi sử dụng các instance **Graviton**, chúng ta cần đảm bảo rằng chỉ các container được xây dựng cho các kiến trúc CPU **ARM** mới được lên lịch trên các instance **Graviton**. Đây là nơi mà **taints** và **tolerations** trở nên hữu ích.