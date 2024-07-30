---
title: "Chạy các pods trên Graviton"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 4.2 </b>"
---

#### Chạy các pods trên Graviton

Sau khi đã tạo ra Node group Graviton của chúng ta, chúng ta sẽ cần cấu hình ứng dụng của mình để tận dụng các thay đổi này. Để làm điều đó, hãy cấu hình ứng dụng của chúng ta để triển khai dịch vụ micro `ui` chỉ trên các nodes là phần của Amazon EKS Managed Node Groups dựa trên Graviton của chúng ta.

Trước khi thực hiện bất kỳ thay đổi nào, hãy kiểm tra cấu hình hiện tại cho các pods UI. Hãy nhớ rằng các pods này đang được điều khiển bởi một bản triển khai kết hợp được gọi là `ui`.

```bash
$ kubectl describe pod --namespace ui --selector app.kubernetes.io/name=ui
Name:             ui-7bdbf967f9-qzh7f
Namespace:        ui
Priority:         0
Service Account:  ui
Node:             ip-10-42-11-43.us-west-2.compute.internal/10.42.11.43
Start Time:       Wed, 09 Nov 2022 16:40:32 +0000
Labels:           app.kubernetes.io/component=service
                  app.kubernetes.io/created-by=eks-workshop
                  app.kubernetes.io/instance=ui
                  app.kubernetes.io/name=ui
                  pod-template-hash=7bdbf967f9
Status:           Running
[....]
Controlled By:  ReplicaSet/ui-7bdbf967f9
Containers:
[...]
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
```

Như dự kiến, ứng dụng đang chạy thành công trên một nodes không bị ô nhiễm. Các pod tương ứng đang trong trạng thái `Running` và chúng ta có thể xác nhận rằng không có tolerations tùy chỉnh nào đã được cấu hình. Lưu ý rằng Kubernetes tự động thêm tolerations cho `node.kubernetes.io/not-ready` và `node.kubernetes.io/unreachable` với `tolerationSeconds=300`, trừ khi bạn hoặc một bộ điều khiển cụ thể đặt những tolerations đó. Những tolerations được tự động thêm này có nghĩa là Pods vẫn được ràng buộc với Nodes trong 5 phút sau khi phát hiện vấn đề.

Hãy cập nhật bản triển khai `ui` của chúng ta để liên kết các pods của nó với Amazon EKS Managed Node Groups bị ô nhiễm của chúng ta. Chúng ta đã tiền cấu hình Amazon EKS Managed Node Groups bị ô nhiễm của mình với một nhãn là `tainted=yes` mà chúng ta có thể sử dụng với một `nodeSelector`. Patch `Kustomize` sau mô tả các thay đổi cần thiết cho cấu hình triển khai của chúng ta để kích hoạt thiết lập này:

```kustomization
modules/fundamentals/mng/graviton/nodeselector-wo-toleration/deployment.yaml
Deployment/ui
```

Để áp dụng các thay đổi Kustomize, hãy chạy lệnh sau:

```bash
$ kubectl apply -k ~/environment/eks-workshop/modules/fundamentals/mng/graviton/nodeselector-wo-toleration/
namespace/ui unchanged
serviceaccount/ui unchanged
configmap/ui unchanged
service/ui unchanged
deployment.apps/ui configured
```

Với những thay đổi mới được thực hiện gần đây, hãy kiểm tra tình trạng triển khai UI của chúng ta:

```bash
$ kubectl --namespace ui rollout status --watch=false deployment/ui
Waiting for deployment "ui" rollout to finish: 1 old replicas are pending termination...
```

Với chiến lược `RollingUpdate` mặc định cho triển khai `ui` của chúng ta, triển khai K8s sẽ chờ cho đến khi pod mới được tạo ra để ở trạng thái `Ready` trước khi chấm dứt pod cũ. Triển khai triển khai dường như đang bị kẹt, vì vậy hãy điều tra sâu hơn:

```bash hook=pending-pod
$ kubectl get pod --namespace ui -l app.kubernetes.io/name=ui
NAME                  READY   STATUS    RESTARTS   AGE
ui-659df48c56-z496x   0/1     Pending   0          16s
ui-795bd46545-mrglh   1/1     Running   0          8m
```

Bằng cách điều tra các pod cá nhân trong namespace `ui`, chúng ta có thể quan sát rằng một pod đang ở trạng thái `Pending`. Đào sâu hơn vào chi tiết `Pending` Pod cung cấp một số thông tin về vấn đề gặp phải.

```bash
$ podname=$(kubectl get pod --namespace ui --field-selector=status.phase=Pending -o json | \
                jq -r '.items[0].metadata.name') && \
                kubectl describe pod $podname -n ui
Name:           ui-659df48c56-z496x
Namespace:      ui
[...]
Node-Selectors:              kubernetes.io/arch=arm64
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  19s   default-scheduler  0/4 nodes

 are available: 1 node(s) had untolerated taint {frontend: true}, 3 node(s) didn't match Pod's node affinity/selector. preemption: 0/4 nodes are available: 4 Preemption is not helpful for scheduling.
```

Các thay đổi của chúng ta được phản ánh trong cấu hình mới của pod `Pending`. Chúng ta có thể thấy rằng chúng ta đã ghim pod vào bất kỳ nodes nào có nhãn `tainted=yes` nhưng điều này đã gây ra một vấn đề mới khi pod của chúng ta không thể được lên lịch (`PodScheduled False`). Một giải thích hữu ích hơn có thể được tìm thấy trong phần `events`:

```
0/4 nodes are available: 1 node(s) had untolerated taint {frontend: true}, 3 node(s) didn't match Pod's node affinity/selector. preemption: 0/4 nodes are available: 4 Preemption is not helpful for scheduling.
```

Để sửa chữa điều này, chúng ta cần thêm một toleration. Hãy đảm bảo rằng triển khai và các pod tương ứng có thể chịu đựng taint `frontend: true`. Chúng ta có thể sử dụng patch `kustomize` dưới đây để thực hiện các thay đổi cần thiết:

```kustomization
modules/fundamentals/mng/graviton/nodeselector-w-toleration/deployment.yaml
Deployment/ui
```

```bash
$ kubectl apply -k ~/environment/eks-workshop/modules/fundamentals/mng/graviton/nodeselector-w-toleration/
namespace/ui unchanged
serviceaccount/ui unchanged
configmap/ui unchanged
service/ui unchanged
deployment.apps/ui configured
$ kubectl --namespace ui rollout status deployment/ui --timeout=120s
```

Kiểm tra pod UI, chúng ta có thể thấy rằng cấu hình bây giờ bao gồm toleration đã được chỉ định (`frontend=true:NoExecute`) và nó đã được lên lịch thành công trên nodes có taint tương ứng. Các lệnh sau có thể được sử dụng cho việc xác thực:

```bash
$ kubectl get pod --namespace ui -l app.kubernetes.io/name=ui
NAME                  READY   STATUS    RESTARTS   AGE
ui-6c5c9f6b5f-7jxp8   1/1     Running   0          29s
```

```bash
$ kubectl describe pod --namespace ui -l app.kubernetes.io/name=ui
Name:         ui-6c5c9f6b5f-7jxp8
Namespace:    ui
Priority:     0
Node:         ip-10-42-10-138.us-west-2.compute.internal/10.42.10.138
Start Time:   Fri, 11 Nov 2022 13:00:36 +0000
Labels:       app.kubernetes.io/component=service
              app.kubernetes.io/created-by=eks-workshop
              app.kubernetes.io/instance=ui
              app.kubernetes.io/name=ui
              pod-template-hash=6c5c9f6b5f
Annotations:  kubernetes.io/psp: eks.privileged
              prometheus.io/path: /actuator/prometheus
              prometheus.io/port: 8080
              prometheus.io/scrape: true
Status:       Running
IP:           10.42.10.225
IPs:
  IP:           10.42.10.225
Controlled By:  ReplicaSet/ui-6c5c9f6b5f
Containers:
  [...]
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
[...]
QoS Class:                   Burstable
Node-Selectors:              kubernetes.io/arch=arm64
Tolerations:                 frontend:NoExecute op=Exists
                             node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
[...]
```

```bash
$ kubectl describe node --selector kubernetes.io/arch=arm64
Name:               ip-10-42-10-138.us-west-2.compute.internal
Roles:              <none>
Labels:             beta.kubernetes.io/instance-type=t4g.medium
                    beta.kubernetes.io/os=linux
                    eks.amazonaws.com/capacityType=ON_DEMAND
                    eks.amazonaws.com/nodegroup=graviton
                    eks.amazonaws.com/nodegroup-image=ami-03e8f91597dcf297b
                    kubernetes.io/arch=arm64
                    kubernetes.io/hostname=ip-10-42-10-138.us-west-2.compute.internal
                    kubernetes.io/os=linux
                    node.kubernetes.io/instance-type=t4g.medium
[...]
Taints:             frontend=true:NoExecute
Unschedulable:      false
[...]
```

Như bạn có thể thấy, pod `ui` bây giờ đang chạy trên Node Groups dựa trên Graviton. Ngoài ra, bạn có thể thấy các Taints trên lệnh `kubectl describe node`, và các Tolerations phù hợp trên lệnh `kubectl describe pod`.

Bạn đã thành công trong việc lên lịch ứng dụng `ui`, có thể chạy trên cả bộ xử lý Intel và ARM, để chạy trên Amazon EKS Managed Node Groups mới dựa trên Graviton chúng ta đã tạo ra ở bước trước đó. Taints và tolerations là một công cụ mạnh mẽ có thể được sử dụng để cấu hình cách các pod được lên lịch lên các nodes, cho dù đó là cho các nodes được cải thiện bởi Graviton/GPU, hoặc cho các cluster Kubernetes đa người sử dụng.