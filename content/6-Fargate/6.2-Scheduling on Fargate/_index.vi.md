---
title: "Scheduling on Fargate"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 6.2 </b>"
---

#### Scheduling on Fargate

Trong hệ thống của chúng ta, dịch vụ `checkout` là một phần không thể thiếu trong quy trình thanh toán của khách hàng. Vậy tại sao dịch vụ này lại chưa được chạy trên **AWS Fargate**? Hãy cùng tìm hiểu qua các nhãn (labels) của nó.

Để kiểm tra, chúng ta sẽ sử dụng lệnh sau trong Kubernetes (k8s):

```bash
$ kubectl get pod -n checkout -l app.kubernetes.io/component=service -o json | jq '.items[0].metadata.labels'
```

Có vẻ như Pod của chúng ta thiếu nhãn `fargate=yes`. Vì vậy, chúng ta sẽ khắc phục điều này bằng cách cập nhật deployment cho dịch vụ đó sao cho phần đặc tả (spec) của Pod bao gồm nhãn cần thiết cho việc lên lịch trên Fargate.

Cập nhật deployment như sau:

```kustomization
modules/fundamentals/fargate/enabling/deployment.yaml
Deployment/checkout
```

Áp dụng kustomization vào cluster:

```bash timeout=220 hook=enabling
$ kubectl apply -k ~/environment/eks-workshop/modules/fundamentals/fargate/enabling
[...]
$ kubectl rollout status -n checkout deployment/checkout --timeout=200s
```

Điều này sẽ khiến cho đặc tả Pod của dịch vụ `checkout` được cập nhật và kích hoạt một deployment mới, thay thế tất cả các Pods. Khi các Pods mới được lập lịch, bộ lên lịch Fargate sẽ so khớp nhãn mới được áp dụng bởi kustomization với hồ sơ mục tiêu của chúng ta và can thiệp để đảm bảo Pod của chúng ta được lập lịch trên tài nguyên do Fargate quản lý.

Làm thế nào để chúng ta xác nhận nó đã hoạt động? Mô tả Pod mới đã được tạo và xem phần `Events`:

```bash
$ kubectl describe pod -n checkout -l fargate=yes
[...]
Events:
  Type     Reason           Age    From               Message
  ----     ------           ----   ----               -------
  Warning  LoggingDisabled  10m    fargate-scheduler  Disabled logging because aws-logging configmap was not found. configmap "aws-logging" not found
  Normal   Scheduled        9m48s  fargate-scheduler  Successfully assigned checkout/checkout-78fbb666b-fftl5 to fargate-ip-10-42-11-96.us-west-2.compute.internal
  Normal   Pulling          9m48s  kubelet            Pulling image "public.ecr.aws/aws-containers/retail-store-sample-checkout:0.4.0"
  Normal   Pulled           9m5s   kubelet            Successfully pulled image "public.ecr.aws/aws-containers/retail-store-sample-checkout:0.4.0" in 43.258137629s
  Normal   Created          9m5s   kubelet            Created container checkout
  Normal   Started          9m4s   kubelet            Started container checkout
```

Các sự kiện từ `fargate-scheduler` cho chúng ta một số cái nhìn về những gì đã xảy ra. Sự kiện chính mà chúng ta quan tâm ở giai đoạn này của lab là sự kiện với lý do `Scheduled`. Kiểm tra kỹ lưỡng cho chúng ta biết tên của thể hiện Fargate được cung cấp cho Pod này, trong trường hợp của ví dụ trên là `fargate-ip-10-42-11-96.us-west-2.compute.internal`.

Chúng ta có thể kiểm tra nút này từ `kubectl` để lấy thêm thông tin về tài nguyên được cung cấp cho Pod này:

```bash
$ NODE_NAME=$(kubectl get pod -n checkout -l app.kubernetes.io/component=service -o json | jq -r '.items[0].spec.nodeName')
$ kubectl describe node $NODE_NAME
Name:               fargate-ip-10-42-11-96.us-west-2.compute.internal
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    eks.amazonaws.com/compute-type=fargate
                    failure-domain.beta.kubernetes.io/region=us-west-2
                    failure-domain.beta.kubernetes.io/zone=us-west-2b
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=ip-10-42-11-96.us-west-2.compute.internal
                    kubernetes.io/os=linux
                    topology.kubernetes.io/region=us-west-2
                    topology.kubernetes.io/zone=us-west-2b
[...]
```

Điều này cung cấp cho chúng ta một số cái nhìn về bản chất của thể hiện tính toán cơ bản:

- Nhãn `eks.amazonaws.com/compute-type` xác nhận rằng một thể hiện Fargate đã được cung cấp.
- Một nhãn khác `topology.kubernetes.io/zone` chỉ định khu vực khả dụng mà pod đang chạy.
- Trong phần `System Info` (không được hiển thị ở trên), chúng ta có thể thấy thể hiện đang chạy Amazon Linux 2, cũng như thông tin phiên bản cho các thành phần hệ thống như `container`, `kubelet` và `kube-proxy`.