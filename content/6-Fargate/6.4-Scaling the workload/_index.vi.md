---
title: "Scaling the workload"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 6.4 </b>"
---

#### Scaling the workload

Một trong những lợi ích nổi bật của **Fargate** đó là mô hình mở rộng quy mô ngang cách đơn giản mà nó mang lại. Khi sử dụng **EC2** cho việc tính toán, việc mở rộng quy mô các **Pods** đòi hỏi phải xem xét không chỉ các **Pods** sẽ được mở rộng quy mô như thế nào mà còn cả phần cứng tính toán phía dưới. Bởi vì **Fargate** tách biệt phần cứng tính toán cơ bản, bạn chỉ cần quan tâm đến việc mở rộng quy mô chính các **Pods**.

Trong các ví dụ mà chúng ta đã xem xét cho đến nay, chỉ sử dụng một bản sao **Pod** duy nhất. Điều gì sẽ xảy ra nếu chúng ta mở rộng quy mô theo cách ngang như chúng ta thường mong đợi trong một kịch bản thực tế? Hãy mở rộng quy mô dịch vụ `checkout` và tìm hiểu:

```yaml
modules/fundamentals/fargate/scaling/deployment.yaml
Deployment/checkout
```

Áp dụng kustomization và chờ đợi cho đến khi việc triển khai hoàn tất:

```bash
$ kubectl apply -k ~/environment/eks-workshop/modules/fundamentals/fargate/scaling
[...]
$ kubectl rollout status -n checkout deployment/checkout --timeout=200s
```

Một khi việc triển khai hoàn tất, chúng ta có thể kiểm tra số lượng **Pods**:

```bash
$ kubectl get pod -n checkout -l app.kubernetes.io/component=service
NAME                        READY   STATUS    RESTARTS   AGE
checkout-585c9b45c7-2c75m   1/1     Running   0          2m12s
checkout-585c9b45c7-c456l   1/1     Running   0          2m12s
checkout-585c9b45c7-xmx2t   1/1     Running   0          40m
```

Mỗi **Pod** này được lên lịch trên một thể hiện **Fargate** riêng biệt. Bạn có thể xác nhận điều này bằng cách thực hiện các bước tương tự như trước đây và xác định node của một **Pod** cụ thể.
