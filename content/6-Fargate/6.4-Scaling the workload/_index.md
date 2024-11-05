---
title: "Scaling the workload"

weight: 4
chapter: false
pre: "<b> 6.4 </b>"
---

Another benefit of Fargate is the simplified horizontal scaling model it offers. When using EC2 for compute, scaling Pods involves considering how not only the Pods will scale but also the underlying compute. Because Fargate abstracts away the underlying compute you only need to be concerned with scaling Pods themselves.

The examples we've looked at so far have only used a single Pod replica. What happens if we scale this out horizontally as we would typically expect in a real life scenario? Let's scale up the `checkout` service and find out:

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

Apply the kustomization and wait for the rollout to complete:

```bash timeout=240
$ kubectl apply -k ~/environment/eks-workshop/modules/fundamentals/fargate/scaling
[...]
$ kubectl rollout status -n checkout deployment/checkout --timeout=200s
```

Once the rollout is complete we can check the number of Pods:

```bash
$ kubectl get pod -n checkout -l app.kubernetes.io/component=service
NAME                        READY   STATUS    RESTARTS   AGE
checkout-585c9b45c7-2c75m   1/1     Running   0          2m12s
checkout-585c9b45c7-c456l   1/1     Running   0          2m12s
checkout-585c9b45c7-xmx2t   1/1     Running   0          40m
```

Each of these Pods is scheduled on a separate Fargate instance. You can confirm this by following similar steps as previously and identifying the node of a given Pod.