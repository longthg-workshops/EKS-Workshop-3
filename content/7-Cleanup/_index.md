---
title: "Cleanup"

weight: 7
chapter: false
pre: "<b> 7. </b>"
---

Make sure you have run the respective clean up instructions for the mechanism you used to provision the lab EKS cluster before proceeding, as instructed in [2.2.1](../2-Prerequiste/2.2-cluster-creation/2.2-eksctl/2.2.1-eksctl/) or [2.2.2](../2-Prerequiste/2.2-cluster-creation/2.2.2-terraform/) (depending on whether you use `eksctl` or `Terraform` to deploy your cluster).

After cleaning up the cluster, run the following command to remove the IDE stack:
```bash
aws cloudformation delete-stack --stack-name eks-workshop-ide
```