---
title: "Resource allocation"

weight: 3
chapter: false
pre: "<b> 6.3 </b>"
---

### Fargate pricing

The primary dimensions of [Fargate pricing](https://aws.amazon.com/fargate/pricing/) is based on CPU and memory, and the amount of resources allocated to a Fargate instance depend on the resource requests specified by the Pod. There is a [documented set](https://docs.aws.amazon.com/eks/latest/userguide/fargate-pod-configuration.html#fargate-cpu-and-memory) of valid CPU and memory combinations for Fargate that should be considered when assessing if a workload is suitable for Fargate.

### See pod resource allocation

We can confirm what resources were provisioned for our Pod from the previous deployment by inspecting its annotations:

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

In this example (above), we can see that the `CapacityProvisioned` annotation shows that we were allocated 0.25 vCPU and 0.5 GB of memory, which is the minimum Fargate instance size. But what if our Pod needs more resources? Luckily Fargate provides a variety of options depending on the resource requests that we can try out.

### Modify pod resource allocation

In the next example we'll increase the amount of resources the `checkout` component is requesting and see how the Fargate scheduler adapts. The kustomization we're going to apply increases the resources requested to 1 vCPU and 2.5G of memory:

```
~/environment/eks-workshop/modules/fundamentals/fargate/sizing/deployment.yaml
```

Kustomization:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: checkout
spec:
  template:
    spec:
      containers:
        - name: checkout
          resources:
            requests:
              cpu: "1"
              memory: 2.5G
            limits:
              memory: 2.5G
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
  replicas: 1
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

```bash timeout=220
$ kubectl apply -k ~/environment/eks-workshop/modules/fundamentals/fargate/sizing
[...]
$ kubectl rollout status -n checkout deployment/checkout --timeout=200s
```

Now, let's check again the resource allocated by Fargate. Based on the changes outlined above, what do you expect to see?

```bash
$ kubectl get pod -n checkout -l app.kubernetes.io/component=service -o json | jq -r '.items[0].metadata.annotations'
{
  "CapacityProvisioned": "1vCPU 3GB",
  "Logging": "LoggingDisabled: LOGGING_CONFIGMAP_NOT_FOUND",
  "kubernetes.io/psp": "eks.privileged",
  "prometheus.io/path": "/metrics",
  "prometheus.io/port": "8080",
  "prometheus.io/scrape": "true"
}
```

The resources requested by the Pod have been rounded up to the nearest Fargate configuration outlined in the valid set of combinations above.