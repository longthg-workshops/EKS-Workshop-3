---
title: "Mở rộng/thu hẹp khối
 công việc"

weight: 4
chapter: false
pre: "<b> 6.4 </b>"
---

#### Scaling the workload

Một trong những lợi ích nổi bật của **Fargate** đó là mô hình mở rộng quy mô ngang cách đơn giản mà nó mang lại. Khi sử dụng **EC2** cho việc tính toán, việc mở rộng quy mô các **Pods** đòi hỏi phải xem xét không chỉ các **Pods** sẽ được mở rộng quy mô như thế nào mà còn cả phần cứng tính toán phía dưới. Bởi vì **Fargate** tách biệt phần cứng tính toán cơ bản, bạn chỉ cần quan tâm đến việc mở rộng quy mô chính các **Pods**.

Trong các ví dụ mà chúng ta đã xem xét cho đến nay, chỉ sử dụng một bản sao **Pod** duy nhất. Điều gì sẽ xảy ra nếu chúng ta mở rộng quy mô theo cách ngang như chúng ta thường mong đợi trong một kịch bản thực tế? Hãy mở rộng quy mô dịch vụ `checkout` và tìm hiểu:

```
~/environment/eks-workshop/modules/fundamentals/fargate/scaling/deployment.yaml
```

Kustomization:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: checkout
spec:
  replicas: 3
```

Deployment yaml:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/created-by: eks-workshop
    app.kubernetes.io/type: app
  name: checkout
  namespace: checkout
spec:
  replicas: 3
  selector:
    matchLabels:
      app.kubernetes.io/component: service
      app.kubernetes.io/instance: checkout
      app.kubernetes.io/name: checkout
  template:
    metadata:
      annotations:
        prometheus.io/path: /metrics
        prometheus.io/port: "8080"
        prometheus.io/scrape: "true"
      labels:
        app.kubernetes.io/component: service
        app.kubernetes.io/created-by: eks-workshop
        app.kubernetes.io/instance: checkout
        app.kubernetes.io/name: checkout
        fargate: yes
    spec:
      containers:
        - envFrom:
            - configMapRef:
                name: checkout
          image: public.ecr.aws/aws-containers/retail-store-sample-checkout:0.4.0
          imagePullPolicy: IfNotPresent
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 3
          name: checkout
          ports:
            - containerPort: 8080
              name: http
              protocol: TCP
          resources:
            limits:
              memory: 2.5G
            requests:
              cpu: "1"
              memory: 2.5G
          securityContext:
            capabilities:
              drop:
                - ALL
            readOnlyRootFilesystem: true
          volumeMounts:
            - mountPath: /tmp
              name: tmp-volume
      securityContext:
        fsGroup: 1000
      serviceAccountName: checkout
      volumes:
        - emptyDir:
            medium: Memory
          name: tmp-volume
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
