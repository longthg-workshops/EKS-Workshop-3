---
title: "Fargate"

weight: 6
chapter: false
pre: "<b> 6. </b>"
---

Prepare your environment for this section:

```bash timeout=400 wait=30
$ prepare-environment fundamentals/fargate
```

This will make the following changes to your lab environment:

- Create an IAM role to be used by Fargate

You can view the Terraform that applies these changes [here](https://github.com/aws-samples/eks-workshop-v2/tree/stable/manifests/modules/fundamentals/fargate/.workshop/terraform).

In the previous module we saw how to provision EC2 compute instances to run Pods in our EKS cluster, and how managed node groups help reduce the operational burden. However, in this model you’re still responsible for the availability, capacity, and maintenance of the underlying infrastructure.

Fargate is a serverless computing technology that reduces your concern about allocating hardware resources to your system. In this section, we will learn how to enable Fargate, then perform scheduling, allocating resources, and scaling workloads running on Fargate.