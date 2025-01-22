---
title: "AWS Graviton Instance"
weight: 1
chapter: false
pre: "<b> 1.1 </b>"
---

**AWS Graviton** processors are built on the [AWS Nitro System](https://aws.amazon.com/ec2/nitro/?p=pm&pd=graviton&z=3). **AWS** created the **AWS Nitro System** so that nearly all compute and memory resources of the server hardware are assigned to your instances. This is achieved by decomposing the **hypervisor** functions and management capabilities from the servers and assigning them to functionally separate hardware and software. This results in better overall performance, unlike traditional virtualization platforms that run the **hypervisor** software on the same physical server as the virtual machines, which means the virtual machines cannot use 100% of the server's resources. **AWS Nitro** systems are supported by popular **Linux** operating systems along with many popular applications and services from **AWS** and **Independent Software Vendors**.

### Multi-architecture Model with Graviton Processors

**AWS Graviton** requires **ARM**-compatible container images, ideally multi-architecture (**ARM64** and **AMD64**) to enable cross-compatibility with both **Graviton** and **x86** instance types.

**Graviton** processors enhance the **EKS** experience for Amazon EKS Managed Node Groups with instances that deliver up to 20% lower prices, up to 40% better price performance, and up to 60% lower power consumption than comparable fifth-generation **x86** instances. Amazon EKS Managed Node Groups **EKS** based on **Graviton** bootstrap an **EC2 Auto Scaling** group with **Graviton** processors.

Adding **Graviton**-based instances to your Amazon EKS Managed Node Groups **EKS** introduces a multi-architecture infrastructure and the need for your application to run on different CPUs. This means that your application code needs to be available in different **Instruction Set Architecture (ISA)** implementations. There are a variety of resources to help teams plan and migrate applications to **Graviton**-based instances. Check out the [Graviton migration plan](https://pages.awscloud.com/rs/112-TZM-766/images/Graviton%20Challenge%20Plan.pdf) and [Porting Advisor for Graviton](https://github.com/aws/porting-advisor-for-graviton) for helpful resources.

![EKS](EKS-Workshop-3/images/4/00011.png?featherlight=false&width=40pc)

The retail store sample web application architecture contains [container images pre-built for both **x86-64** and **ARM64** CPU architectures](https://gallery.ecr.aws/aws-containers/retail-store-sample-ui).

When using **Graviton** instances, we need to ensure that only containers built for **ARM** CPU architectures are scheduled on **Graviton** instances. This is where **taints** and **tolerations** come in handy.