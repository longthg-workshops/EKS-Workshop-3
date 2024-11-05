---
title: "Amazon Elastic Kubernetes Service - Computing"

weight: 1
chapter: false
---

# Amazon Elastic Kubernetes Service - Computing

### Overview

An EKS cluster contains one or more EC2 nodes that Pods are scheduled on. EKS nodes run in your AWS account and connect to the control plane of your cluster through the cluster API server endpoint.

EKS nodes can be:

- **Amazon EC2 instances:** You're billed for them based on EC2 prices. (See [Amazon EC2 pricing](https://aws.amazon.com/ec2/pricing/)). You can deploy one or more nodes into a node group. A node group is one or more EC2 instances that are deployed in an EC2 Auto Scaling group.

- **AWS Fargate:** A serverless computing service that provides on-demand, right-sized compute capacity for containers. With AWS Fargate, you don't have to provision, configure, or scale groups of EC2 instances on your own to run containers. You also don't need to choose server types, decide when to scale your node groups, or optimize cluster packing. Instead you can control which pods start on Fargate and how they run with Fargate profiles.

### Lab Agenda
In this lab, we're going to explore a few computing concepts and resources on EKS, including:

- Managed Node Group
- Managing and upgrading AMIs
- Graviton ARM Instances
- Cost saving with Spot Instance
- Deploy your pods on AWS Fargate
