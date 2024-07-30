---
title: "Đa dạng hóa loại Instance"
date: "2024-04-05"
weight: 1
chapter: false
pre: "<b> 5.1 </b>"
---

#### Đa dạng hóa loại Instance

[Amazon EC2 Spot Instances](https://aws.amazon.com/ec2/spot/) cung cấp khả năng tính toán dư thừa có sẵn trong Đám mây AWS với mức giảm giá đáng kể so với giá On-Demand. EC2 có thể ngắt Spot Instances với thông báo trước hai phút khi EC2 cần lại khả năng tính toán. Bạn có thể sử dụng Spot Instances cho các ứng dụng có tính chịu lỗi và linh hoạt khác nhau. Một số ví dụ là phân tích dữ liệu, các khối công việc được container hóa, tính toán hiệu suất cao (HPC), máy chủ web không trạng thái, kết xuất, CI/CD, và các khối công việc thử nghiệm và phát triển khác.

Một trong những phương pháp tốt nhất để áp dụng thành công Spot Instances là thực hiện **đa dạng hóa Spot Instance** như một phần của cấu hình của bạn. Đa dạng hóa Spot Instance giúp tiếp nhận khả năng từ nhiều group Spot Instance khác nhau, cả cho việc mở rộng và thay thế Spot Instances có thể nhận thông báo chấm dứt Spot Instance. Một group Spot Instance là một tập hợp các EC2 instances không sử dụng với cùng một loại Instance, hệ điều hành và AZ (ví dụ: `m5.large` trên Red Hat Enterprise Linux ở `us-east-1a`).

#### Cluster Autoscaler với Đa dạng hóa Spot Instance

Cluster Autoscaler là một công cụ tự động điều chỉnh kích thước của cluster Kubernetes khi có các pod không chạy trong cluster do tài nguyên không đủ (Mở rộng) hoặc có các nút trong cluster đã được sử dụng không đủ trong một khoảng thời gian (Thu nhỏ).


Khi sử dụng Spot Instances với [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler) có một số điều cần [xem xét](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md). Một điều quan trọng cần xem xét là, mỗi Nhóm Tăng tự động nên được tạo từ các loại instance cung cấp khả năng gần như bằng nhau. Cluster Autoscaler sẽ cố gắng xác định các tài nguyên CPU, bộ nhớ và GPU được cung cấp bởi một Nhóm Tăng tự động dựa trên ghi đè đầu tiên được cung cấp trong Chính sách Các Loại Instances Kết hợp của một Nhóm Tăng tự động. Nếu có bất kỳ ghi đè nào được tìm thấy, chỉ loại instance đầu tiên được tìm thấy sẽ được sử dụng. Xem [Sử dụng Chính Sách Các Loại Instances Kết hợp và Spot Instances](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md#Using-Mixed-Instances-Policies-and-Spot-Instances) để biết chi tiết.


Khi áp dụng các phương pháp đa dạng hóa Spot vào EKS và các cluster K8s trong khi sử dụng Cluster Autoscaler để tự động mở rộng khả năng, chúng ta phải thực hiện đa dạng hóa theo cách tuân thủ các chế độ hoạt động được mong đợi của Cluster Autoscaler.

Chúng ta có thể đa dạng hóa các group Spot Instance bằng hai chiến lược:

- Bằng cách tạo ra nhiều node groups, mỗi group có kích thước khác nhau. Ví dụ, một node groups có kích thước 4 vCPU và 16GB RAM, và một node groups khác có 8 vCPU và 32GB RAM.
- Bằng cách thực hiện đa dạng hóa instance trong các node groups, bằng cách chọn một sự kết hợp của các loại instance và các gia đình khác nhau từ các group Spot Instance khác nhau mà đáp ứng các tiêu chí cùng về vCPU và bộ nhớ.

Trong bài thực hành này, chúng ta sẽ giả sử rằng các node groups của chúng ta nên được cung cấp với các loại instance có 2 vCPU và 4GiB bộ nhớ.

Chúng ta sẽ sử dụng **[amazon-ec2-instance-selector](https://github.com/aws/amazon-ec2-instance-selector)** để giúp chọn các loại instance và gia đình phù hợp có đủ số lượng vCPU và bộ nhớ.

Có hơn 350 loại instance khác nhau có sẵn trên EC2 có thể làm cho quá trình chọn loại instance phù hợp khó khăn `amazon-ec2-instance-selector` giúp bạn chọn các loại instance tương thích cho ứng dụng của bạn chạy trên. Giao diện dòng lệnh có thể được truyền các tiêu chí tài nguyên như vcpus, bộ nhớ, hiệu suất mạng, và nhiều hơn nữa và sau đó trả về các loại instance phù hợp có sẵn.

Công cụ CLI đã được cài đặt sẵn trong IDE của bạn:

```bash
$ ec2-instance-selector --version
```

Bây giờ bạn đã cài đặt ec2-instance-selector, bạn có thể chạy `ec2-instance-selector --help` để hiểu cách bạn có thể sử dụng nó để chọn các loại instance phù hợp với yêu cầu công việc của bạn. Vì mục đích của bài thực hành này, chúng ta cần trước tiên lấy một group các loại instance đáp ứng mục tiêu của chúng tôi là 2 vCPU và 4 GB RAM.

Chạy lệnh sau để lấy danh sách các loại instance.

```bash
$ ec2-instance-selector --vcpus 2 --memory 4 --gpus 0 --current-generation \
  -a x86_64 --deny-list 't.*' --output table-wide
Instance Type  VCPUs   Mem (GiB)  Hypervisor  Current Gen  Hibernation Support  CPU Arch  Network Performance
-------------  -----   ---------  ----------  -----------  -------------------  --------  -------------------
c5.large       2       4          nitro       true         true                 x86_64    Up to 10 Gigabit
c5a.large      2       4          nitro       true         false                x86_64    Up to 10 Gigabit
c5ad.large     2       4          nitro       true         false                x86_64    Up to 10 Gigabit
c5d.large      2       4          nitro       true         true                 x86_64    Up to 10 Gigabit
c6a.large      2       4          nitro       true         false                x86_64    Up to 12.5 Gigabit
c6i.large      2       4          nitro       true         true                 x86_64    Up to 12.5 Gigabit
c6id.large     2       4          nitro       true         true                 x86_64    Up to 12.5 Gigabit
c6in.large     2       4          nitro       true         false                x86_64    Up to 25 Gigabit
c7a.large      2       4          nitro       true         false                x86_64    Up to 12.5 Gigabit
c7i.large      2       4          nitro       true         false                x86_64    Up to 12.5 Gigabit
```

Chúng ta sẽ sử dụng các loại instance này khi chúng ta định nghĩa node groups của mình trong phần tiếp theo.

Bên trong, `ec2-instance-selector` đang gọi đến [DescribeInstanceTypes](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeInstanceTypes.html) cho khu vực cụ thể và lọc các loại instance dựa trên các tiêu chí được chọn trong dòng lệnh, trong trường hợp của chúng ta, chúng ta đã lọc các loại instance đáp ứng các tiêu chí sau:

- Các loại instance không có GPU
- Kiến trúc x86_64 (không có các loại instance ARM như A1 hoặc các loại instance m6g chẳng hạn)
- Các loại instance có 2 vCPU và 4 GB RAM
- Các loại instance thế hệ hiện tại (từ thế hệ thứ 4 trở đi)
- Các loại instance không đáp ứng biểu thức chính quy `t.*` để loại bỏ các loại instance burstable

Khối công việc của bạn có thể có các ràng buộc khác mà bạn nên xem xét khi chọn loại instance. Ví dụ. Các loại instance **t2** và **t3** là [loại burstable](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-performance-instances.html) và có thể không phù hợp cho các khối công việc có hạn chế về CPU yêu cầu độ chính xác của CPU. Các loại instance như m5**a** là [loại AMD Instances](https://aws.amazon.com/ec2/amd/), nếu khối công việc của bạn nhạy cảm với sự khác biệt số liệu (ví dụ: tính toán rủi ro tài chính, mô phỏng công nghiệp) việc kết hợp các loại instance này có thể không phù hợp.
