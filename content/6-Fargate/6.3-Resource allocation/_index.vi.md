---
title: "Resource allocation"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 6.3 </b>"
---

#### Resource allocation

Khi xem xét giải pháp container hóa cho các ứng dụng, **AWS Fargate** và **Kubernetes** là hai công nghệ được nhiều người chọn lựa. Đặc biệt, việc hiểu rõ cách thức tính toán chi phí của Fargate và cách cấu hình tài nguyên cho Pod trong Kubernetes là cực kỳ quan trọng để tối ưu hóa hiệu suất và chi phí.

#### Điểm Đến của Chi Phí Fargate

Theo [trang giá của Fargate](https://aws.amazon.com/fargate/pricing/), **chi phí Fargate** chủ yếu dựa trên CPU và bộ nhớ được sử dụng. Số lượng tài nguyên được cấp phát cho một thể hiện Fargate phụ thuộc vào yêu cầu tài nguyên được chỉ định bởi Pod. Có một [bộ cấu hình đã được tài liệu](https://docs.aws.amazon.com/eks/latest/userguide/fargate-pod-configuration.html#fargate-cpu-and-memory) về các tổ hợp CPU và bộ nhớ hợp lệ cho Fargate, điều này cần được cân nhắc khi đánh giá một tải công việc có phù hợp với Fargate hay không.

Chúng ta có thể xác định tài nguyên nào đã được cung cấp cho Pod của mình từ lần triển khai trước bằng cách kiểm tra các annotation của nó:

```bash
$ kubectl get pod -n checkout -l app.kubernetes.io/component=service -o json | jq -r '.items[0].metadata.annotations'
{
  "CapacityProvisioned": "0.25vCPU 0.5GB",
  "Logging": "LoggingDisabled: LOGGING_CONFIGMAP_NOT_FOUND",
  "kubernetes.io/psp": "eks.privileged",
  "prometheus.io/path": "/metrics",
  "prometheus.io/port": "8080",
  "prometheus.io/scrape": "true"
}
```

Trong ví dụ trên, ta có thể thấy rằng annotation `CapacityProvisioned` chỉ ra rằng chúng tôi đã được cấp phát 0.25 vCPU và 0.5 GB bộ nhớ, đó là kích thước thể hiện Fargate tối thiểu. Nhưng nếu Pod của chúng tôi cần nhiều tài nguyên hơn thì sao? May mắn thay, Fargate cung cấp nhiều lựa chọn tùy thuộc vào yêu cầu tài nguyên mà chúng tôi có thể thử nghiệm.

Trong ví dụ tiếp theo, chúng tôi sẽ tăng số lượng tài nguyên mà thành phần `checkout` yêu cầu và xem cách bộ lập lịch Fargate thích nghi. Kustomization mà chúng tôi sẽ áp dụng tăng tài nguyên yêu cầu lên 1 vCPU và 2.5G bộ nhớ:

```kustomization
modules/fundamentals/fargate/sizing/deployment.yaml
Deployment/checkout
```

Áp dụng kustomization và chờ cho quá trình triển khai hoàn tất:

```bash timeout=220
$ kubectl apply -k ~/environment/eks-workshop/modules/fundamentals/fargate/sizing
[...]
$ kubectl rollout status -n checkout deployment/checkout --timeout=200s
```

Bây giờ, hãy kiểm tra lại tài nguyên được Fargate cấp phát. Dựa trên những thay đổi được nêu trên, bạn mong đợi thấy gì?

```bash
$ kubectl get pod -n checkout -l app.kubernetes.io/component=service -

o json | jq -r '.items[0].metadata.annotations'
{
  "CapacityProvisioned": "1vCPU 3GB",
  "Logging": "LoggingDisabled: LOGGING_CONFIGMAP_NOT_FOUND",
  "kubernetes.io/psp": "eks.privileged",
  "prometheus.io/path": "/metrics",
  "prometheus.io/port": "8080",
  "prometheus.io/scrape": "true"
}
```

Tài nguyên được yêu cầu bởi Pod đã được làm tròn lên tới cấu hình Fargate gần nhất được mô tả trong bộ tổ hợp hợp lệ trên.
