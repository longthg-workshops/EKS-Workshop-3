---
title: "Spot instances"

weight: 5
chapter: false
pre: "<b> 5. </b>"
---

Before you start Prepare your environment for this section.

```bash
$ prepare-environment fundamentals/mng/spot
```

All of our existing compute nodes are using On-Demand capacity. However, there are multiple "[purchase options](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-purchasing-options.html)" available to EC2 customers for running their EKS workloads, including Spot Instances.

In this lab we'll look at how we can leverage EC2 Spot capacity with EKS managed node groups.