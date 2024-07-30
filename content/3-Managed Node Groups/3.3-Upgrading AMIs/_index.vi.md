---
title: "Nâng cấp AMI"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 3.3 </b>"
---

#### Nâng cấp AMI

[Amazon EKS Optimized Amazon Linux AMI](https://docs.aws.amazon.com/eks/latest/userguide/eks-optimized-amis.html) được xây dựng trên nền tảng **Amazon Linux 2** và được cấu hình để phục vụ như là base image cho các node của **Amazon EKS**. Được coi là một thực hành tốt nhất khi sử dụng phiên bản mới nhất của **EKS-Optimized AMI** khi bạn thêm node vào một cluster **EKS**, vì các bản phát hành mới bao gồm các bản vá và cập nhật bảo mật cho **Kubernetes**. Việc nâng cấp các node đã được cung cấp trước đó trong cluster **EKS** cũng rất quan trọng.

Amazon EKS Managed Node Groups cung cấp khả năng tự động hóa việc cập nhật **AMI** đang được sử dụng bởi các node mà nó quản lý. Nó sẽ tự động dỡ bỏ node bằng cách sử dụng **Kubernetes API** và tuân thủ [Pod disruption budgets](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) mà bạn đặt cho các **Pod** của mình để đảm bảo rằng ứng dụng của bạn luôn sẵn sàng.

Quá trình nâng cấp Amazon EKS Managed Node Groups của **Amazon EKS** gồm 4 giai đoạn:

**Thiết lập**:

- Tạo ra một phiên bản **Amazon EC2 Launch Template** mới được liên kết với **Auto Scaling group** với **AMI** mới nhất
- Chỉ định **Auto Scaling group** của bạn sử dụng phiên bản mới nhất của launch template
- Xác định số lượng node tối đa để nâng cấp song song sử dụng thuộc tính `updateconfig` cho Node Group.

**Tăng quy mô**:

- Trong quá trình nâng cấp, các node nâng cấp sẽ được triển khai trong cùng một AZ với những node đang được nâng cấp
- Tăng kích thước tối đa và kích thước mong muốn của **Auto Scaling Group** để hỗ trợ các node bổ sung
- Sau khi tăng quy mô **Auto Scaling Group**, nó sẽ kiểm tra xem các node sử dụng cấu hình mới nhất có hiện diện trong Node Group hay không.
- Áp dụng một `eks.amazonaws.com/nodegroup=unschedulable:NoSchedule` taint trên mọi node trong Node Group không có nhãn mới nhất. Điều này ngăn không cho các node đã được cập nhật từ một lần cập nhật trước đó bị gắn nhãn.

**Nâng cấp**:

- Chọn ngẫu nhiên một node và dỡ bỏ các **Pod** khỏi node đó.
- Chuyển trạng thái node sang cordon sau khi mọi **Pod** đã được dỡ bỏ và đợi 60 giây
- Gửi yêu cầu hủy bỏ tới **Auto Scaling Group** cho node đã được cordon.
- Áp dụng tương tự cho tất cả các node thuộc Amazon EKS Managed Node Groups, đảm bảo không có node nào sử dụng phiên bản cũ hơn

**Giảm quy mô**:

- Giai đoạn giảm quy mô sẽ giảm kích thước tối đa

 và kích thước mong muốn của **Auto Scaling group** xuống một đơn vị cho đến khi các giá trị trở về như trước khi bắt đầu quá trình cập nhật.

Để tìm hiểu thêm về hành vi cập nhật Amazon EKS Managed Node Groups, hãy xem [các giai đoạn cập nhật Amazon EKS Managed Node Groups](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-update-behavior.html).

### Nâng cấp một Amazon EKS Managed Node Groups

Việc nâng cấp Node Group sẽ mất ít nhất 10 phút, chỉ thực hiện phần này nếu bạn có đủ thời gian

Cluster **EKS** đã được cung cấp cho bạn cố tình có các Amazon EKS Managed Node Groups không chạy phiên bản **AMI** mới nhất. Bạn có thể xem phiên bản **AMI** mới nhất là gì bằng cách truy vấn **SSM**:

```bash
$ EKS_VERSION=$(aws eks describe-cluster --name $EKS_CLUSTER_NAME --query "cluster.version" --output text)
$ aws ssm get-parameter --name /aws/service/eks/optimized-ami/$EKS_VERSION/amazon-linux-2/recommended/image_id --region $AWS_REGION --query "Parameter.Value" --output text
ami-0fcd72f3118e0dd88
```

Khi bạn khởi tạo một cập nhật Amazon EKS Managed Node Groups, **Amazon EKS** sẽ tự động cập nhật các node của bạn, hoàn thành các bước được liệt kê ở trên. Nếu bạn đang sử dụng một **Amazon EKS Optimized AMI**, **Amazon EKS** sẽ tự động áp dụng các bản vá bảo mật mới nhất và cập nhật hệ điều hành cho các node của bạn như một phần của phiên bản phát hành **AMI** mới nhất.

Bạn có thể khởi tạo một cập nhật của Amazon EKS Managed Node Groups được sử dụng để lưu trữ ứng dụng mẫu của chúng ta như sau:

```bash
$ aws eks update-nodegroup-version --cluster-name $EKS_CLUSTER_NAME --nodegroup-name $EKS_DEFAULT_MNG_NAME
```

Bạn có thể theo dõi hoạt động trên các node bằng cách sử dụng `kubectl`:

```bash test=false
$ kubectl get nodes --watch
```

Nếu bạn muốn đợi cho đến khi **MNG** được cập nhật, bạn có thể chạy lệnh sau:

```bash timeout=2400
$ aws eks wait nodegroup-active --cluster-name $EKS_CLUSTER_NAME --nodegroup-name $EKS_DEFAULT_MNG_NAME
```

Một khi quá trình này hoàn tất, bạn có thể tiếp tục bước tiếp theo.
