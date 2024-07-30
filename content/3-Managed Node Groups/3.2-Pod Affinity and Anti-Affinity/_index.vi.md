---
title: "Pod Affinity and Anti-Affinity"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 3.2 </b>"
---

#### Pod Affinity và Anti-Affinity

Các Pods có thể được hạn chế chạy trên các nodes cụ thể hoặc dưới các điều kiện cụ thể. Điều này có thể bao gồm các trường hợp bạn chỉ muốn một pod ứng dụng chạy trên mỗi node hoặc muốn ghép các pods lại với nhau trên một node. Ngoài ra, khi sử dụng node affinity, các pods có thể có các hạn chế ưu tiên hoặc bắt buộc.

Trong bài học này, chúng ta sẽ tập trung vào inter-pod affinity và anti-affinity bằng cách lên lịch các pods `checkout-redis` chạy chỉ một instance trên mỗi node và lên lịch các pods `checkout` chỉ chạy một instance của nó trên các nodes nơi có pod `checkout-redis` tồn tại. Điều này sẽ đảm bảo rằng các pods caching (`checkout-redis`) của chúng ta chạy cùng địa phương với một instance pod `checkout` để đạt hiệu suất tốt nhất.

Điều đầu tiên chúng ta muốn làm là xem các pods `checkout` và `checkout-redis` đang chạy:

```bash
$ kubectl get pods -n checkout
NAME                              READY   STATUS    RESTARTS   AGE
checkout-698856df4d-vzkzw         1/1     Running   0          125m
checkout-redis-6cfd7d8787-kxs8r   1/1     Running   0          127m
```

Chúng ta có thể thấy cả hai ứng dụng đều có một pod đang chạy trong cluster. Bây giờ, hãy tìm hiểu chúng đang chạy ở đâu:

```bash
$ kubectl get pods -n checkout \
  -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.nodeName}{"\n"}'
checkout-698856df4d-vzkzw       ip-10-42-11-142.us-west-2.compute.internal
checkout-redis-6cfd7d8787-kxs8r ip-10-42-10-225.us-west-2.compute.internal
```

Dựa vào kết quả trên, pod `checkout-698856df4d-vzkzw` đang chạy trên node `ip-10-42-11-142.us-west-2.compute.internal` và pod `checkout-redis-6cfd7d8787-kxs8r` đang chạy trên node `ip-10-42-10-225.us-west-2.compute.internal`.


Trong môi trường của bạn, các pods có thể đang chạy trên cùng một node ban đầu


Hãy thiết lập một chính sách `podAffinity` và `podAntiAffinity` trong deployment **checkout** để đảm bảo một pod `checkout` chạy trên mỗi node, và nó chỉ sẽ chạy trên các nodes có pod `checkout-redis` đang chạy. Chúng ta sẽ sử dụng `requiredDuringSchedulingIgnoredDuringExecution` để làm cho điều này trở thành yêu cầu, thay vì một hành vi được ưu tiên.

Kustomization sau thêm một phần `affinity` vào deployment **checkout** chỉ định cả chính sách **podAffinity** và **podAntiAffinity**:

```kustomization
modules/fundamentals/affinity/checkout/checkout.yaml
Deployment/checkout
```

Để thực hiện thay đổi, chạy lệnh sau để sửa đổi deployment **checkout** trong cluster của bạn:

```bash
$ kubectl delete -n checkout deployment checkout
$ kubectl apply -k ~/environment/eks-workshop/modules/fundamentals/affinity/checkout/
namespace/checkout unchanged
serviceaccount/checkout unchanged
configmap/checkout unchanged
service/checkout unchanged
service/checkout-redis unchanged
deployment.apps/checkout configured
deployment.apps/checkout-redis unchanged
$ kubectl rollout status deployment/checkout \
  -n checkout --timeout 180s
```

Phần **podAffinity** đảm bảo rằng một pod `checkout-redis` đã đang chạy trên node — điều này là vì chúng ta có thể giả định rằng pod `checkout` cần `checkout-redis` để chạy đúng cách. Phần **podAntiAffinity** yêu cầu rằng không có pods `checkout` nào đã đang chạy trên node bằng cách khớp với nhãn **`app.kubernetes.io/component=service`**. Bây giờ, hãy mở rộng deployment để kiểm tra cấu hình đang hoạt động:

```bash
$ kubectl scale -n checkout deployment/checkout --replicas 2
```

Bây giờ xác nhận mỗi pod đang chạy ở đâu:

```bash
$ kubectl get pods -n checkout \
  -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.nodeName}{"\n"}'
checkout-6c7c9cdf4f-p5p6q       ip-10-42-10-120.us-west-2.compute.internal
checkout-6c7c9cdf4f-wwkm4
checkout-redis-6cfd7d8787-gw59j ip-10-42-10-120.us-west-2.compute.internal
```

Trong ví dụ này, pod `checkout` đầu tiên chạy trên cùng pod với pod checkout-redis hiện tại, vì nó đáp ứng quy tắc **podAffinity** mà chúng ta đã đặt. Cái thứ hai vẫn đang chờ, bởi vì quy tắc **podAntiAffinity** mà chúng ta đã định nghĩa không cho phép hai pod checkout khởi động trên cùng một node. Do node thứ hai không có pod `checkout-redis` đang chạy, nó sẽ ở trạng thái chờ.

Tiếp theo, chúng ta sẽ mở rộng `checkout-redis` thành hai instances cho hai nodes của chúng ta, nhưng trước hết hãy sửa đổi chính sách deployment `checkout-redis` để phân tán các instances `checkout-redis` của chúng ta qua mỗi node. Để làm điều này, chúng ta chỉ cần tạo một quy tắc **podAntiAffinity**.

```kustomization
modules/fundamentals/affinity/checkout-redis/checkout-redis.yaml
Deployment/checkout-redis
```

Áp dụng quy tắc với lệnh sau:

```bash
$ kubectl delete -n checkout deployment checkout-redis
$ kubectl apply -k ~/environment/eks-workshop/modules/fundamentals/affinity/checkout-redis/
namespace/checkout unchanged
serviceaccount/checkout unchanged
configmap/checkout unchanged
service/checkout unchanged
service/checkout-redis unchanged
deployment.apps/checkout unchanged
deployment.apps/checkout-redis configured
$ kubectl rollout status deployment/checkout-redis \
  -n checkout --timeout 180s
```

Phần **podAntiAffinity** yêu cầu rằng không có pods `checkout-redis` nào đã đang chạy trên node bằng cách khớp với nhãn **`app.kubernetes.io/component=redis`**.

```bash
$ kubectl scale -n checkout deployment/checkout-redis --replicas 2
```

Kiểm tra các pods đang chạy để xác minh rằng giờ đây có hai cái của mỗi loại đang chạy:

```bash
$ kubectl get pods -n checkout
NAME                             READY   STATUS    RESTARTS   AGE
checkout-5b68c8cddf-6ddwn        1/1     Running   

 0         4m14s
checkout-5b68c8cddf-rd7xf        1/1     Running    0         4m12s
checkout-redis-7979df659-cjfbf   1/1     Running    0         19s
checkout-redis-7979df659-pc6m9   1/1     Running    0         22s
```

Chúng ta cũng có thể xác minh xem các pods đang chạy ở đâu để đảm bảo các chính sách **podAffinity** và **podAntiAffinity** đang được tuân thủ:

```bash
$ kubectl get pods -n checkout \
  -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.nodeName}{"\n"}'
checkout-5b68c8cddf-bn8bp       ip-10-42-11-142.us-west-2.compute.internal
checkout-5b68c8cddf-clnps       ip-10-42-12-31.us-west-2.compute.internal
checkout-redis-7979df659-57xcb  ip-10-42-11-142.us-west-2.compute.internal
checkout-redis-7979df659-r7kkm  ip-10-42-12-31.us-west-2.compute.internal
```

Mọi thứ có vẻ tốt với việc lên lịch các pod, nhưng chúng ta có thể xác minh thêm bằng cách mở rộng pod `checkout` một lần nữa để xem pod thứ ba sẽ được triển khai ở đâu:

```bash
$ kubectl scale --replicas=3 deployment/checkout --namespace checkout
```

Nếu chúng ta kiểm tra các pods đang chạy, chúng ta có thể thấy rằng pod `checkout` thứ ba đã được đặt trong trạng thái Pending vì hai node đã có pod được triển khai và node thứ ba không có pod `checkout-redis` đang chạy.

```bash
$ kubectl get pods -n checkout
NAME                             READY   STATUS    RESTARTS   AGE
checkout-5b68c8cddf-bn8bp        1/1     Running   0          4m59s
checkout-5b68c8cddf-clnps        1/1     Running   0          6m9s
checkout-5b68c8cddf-lb69n        0/1     Pending   0          6s
checkout-redis-7979df659-57xcb   1/1     Running   0          35s
checkout-redis-7979df659-r7kkm   1/1     Running   0          2m10s
```

Hãy kết thúc phần này bằng cách loại bỏ pod đang chờ:

```bash
$ kubectl scale --replicas=2 deployment/checkout --namespace checkout
```
