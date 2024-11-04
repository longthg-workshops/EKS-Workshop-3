---
title: "Giới Hạn Tài Nguyên"

weight: 6
chapter: false
pre: "<b> 1.1.6 </b>"
---

Trong phần này, chúng ta sẽ xem xét về **Giới Hạn Tài Nguyên**.

Hãy xem xét một cụm **Kubernetes** gồm 3 nút. Mỗi nút có một tập hợp các tài nguyên **CPU**, **Bộ nhớ** và **Ổ đĩa** có sẵn.

Nếu không có đủ tài nguyên nào trên các nút, **Kubernetes** sẽ treo việc lên lịch cho pod. Bạn sẽ thấy pod ở trạng thái đang chờ. Nếu bạn xem các sự kiện, bạn sẽ thấy lý do là **CPU** không đủ.

### Yêu Cầu Tài Nguyên

Mặc định, **K8s** giả định rằng một pod hoặc container trong một pod yêu cầu 0.5 **CPU** và 256Mi của **bộ nhớ**. Điều này được biết đến là **Yêu Cầu Tài Nguyên** cho một container.

Nếu ứng dụng của bạn trong pod yêu cầu nhiều hơn các tài nguyên mặc định, bạn cần phải thiết lập chúng trong tệp định nghĩa pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
 containers:
 - name: simple-webapp-color
   image: simple-webapp-color
   ports:
    - containerPort:  8080
   resources:
     requests:
      memory: "1Gi"
      cpu: "1"
```

### Giới Hạn Tài Nguyên

Mặc định, **K8s** thiết lập giới hạn tài nguyên là 1 **CPU** và 512Mi của **bộ nhớ**.

Bạn có thể thiết lập giới hạn tài nguyên trong tệp định nghĩa pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
 containers:
 - name: simple-webapp-color
   image: simple-webapp-color
   ports:
    - containerPort:  8080
   resources:
     requests:
      memory: "1Gi"
      cpu: "1"
     limits:
       memory: "2Gi"
       cpu: "2"
```

Ghi chú: Hãy nhớ **Yêu Cầu** và **Giới Hạn** cho các tài nguyên được thiết lập cho mỗi container trong pod.

### Vượt Quá Giới Hạn

**_Điều gì xảy ra khi một pod cố gắng vượt qua các tài nguyên vượt quá giới hạn của nó ?_**

Nếu một pod vượt quá giới hạn tài nguyên của nó, Kubernetes sẽ thực hiện hành động để ngăn pod tiêu thụ quá nhiều tài nguyên. Hành động cụ thể mà Kubernetes thực hiện sẽ phụ thuộc vào loại tài nguyên đang bị vượt giới hạn.

Nếu một pod vượt quá giới hạn CPU dành cho nó, Kubernetes sẽ chỉ "bóp" hiệu năng pod. Điều này có nghĩa là pod sẽ được cung cấp ít thời gian CPU hơn, dẫn đến giảm hiệu năng và tốc độ xử lý của pod.

Trong khi đó, nếu một pod vượt quá giới hạn bộ nhớ của nó, Kubernetes sẽ tiêu hủy pod với Lý do `OOMKilled` (Out of memory - Hết bộ nhớ). Một pod có thể không phản hồi hoặc thậm chí bị sập khi việc sử dụng bộ nhớ vượt tầm kiểm soát.

### Tài liệu tham khảo
[https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
